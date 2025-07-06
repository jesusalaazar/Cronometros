# Cronometros
Múltiples cronómetros de fácil manejo
import React, { useState, useEffect, useRef } from 'react';

// SVG Icons (replacing lucide-react imports for self-containment)
const PlayIcon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><polygon points="5 3 19 12 5 21 5 3"/></svg>
);
const PauseIcon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/></svg>
);
const Edit2Icon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M17 3a2.85 2.85 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5Z"/></svg>
);
const ChevronRightIcon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m9 18 6-6-6-6"/></svg>
);
const RotateCcwIcon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.76 2.75L3 8"/><path d="M3 3v5h5"/></svg>
);
const PlusIcon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M12 5v14"/><path d="M5 12h14"/></svg>
);
const XIcon = ({ size = 24, className = '' }) => (
  <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>
);

// Helper function to format milliseconds into HH:MM:SS.ms
const formatTime = (milliseconds) => {
  const totalSeconds = Math.floor(milliseconds / 1000);
  const ms = Math.floor((milliseconds % 1000) / 10); // Get milliseconds to 2 digits

  const hours = Math.floor(totalSeconds / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;

  return (
    String(hours).padStart(2, '0') + ':' +
    String(minutes).padStart(2, '0') + ':' +
    String(seconds).padStart(2, '0') + '.' +
    String(ms).padStart(2, '0')
  );
};

const StopwatchItem = ({ stopwatch, updateStopwatchState }) => {
  const { id, name, isRunning, elapsedTime, lapTime, laps, isEditingName } = stopwatch;
  const [isExpanded, setIsExpanded] = useState(false); // State to control expansion of lap history
  const nameInputRef = useRef(null); // Ref for the name input field

  // Focus on the input when editing starts
  useEffect(() => {
    if (isEditingName && nameInputRef.current) {
      nameInputRef.current.focus();
    }
  }, [isEditingName]);

  const handleStartStop = () => {
    updateStopwatchState(id, (prevSw) => {
      const newIsRunning = !prevSw.isRunning;
      if (newIsRunning) {
        prevSw.startTimeRef.current = Date.now() - prevSw.elapsedTime;
        prevSw.lapStartTimeRef.current = Date.now() - prevSw.lapTime;
      }
      return { ...prevSw, isRunning: newIsRunning };
    });
  };

  const handleLap = () => {
    updateStopwatchState(id, (prevSw) => {
      if (prevSw.isRunning) {
        const currentLapDuration = Date.now() - prevSw.lapStartTimeRef.current;
        return {
          ...prevSw,
          laps: [...prevSw.laps, { id: prevSw.laps.length + 1, time: currentLapDuration }],
          lapTime: 0,
          lapStartTimeRef: { current: Date.now() }, // Reset ref for new lap
        };
      }
      return prevSw;
    });
  };

  const handleReset = () => {
    updateStopwatchState(id, (prevSw) => {
      clearInterval(prevSw.intervalRef.current); // Clear existing interval
      return {
        ...prevSw,
        isRunning: false,
        elapsedTime: 0,
        lapTime: 0,
        laps: [],
        startTimeRef: { current: 0 },
        lapStartTimeRef: { current: 0 },
      };
    });
  };

  const handleNameChange = (e) => {
    updateStopwatchState(id, (prevSw) => ({ ...prevSw, name: e.target.value }));
  };

  const handleToggleEditName = () => {
    updateStopwatchState(id, (prevSw) => ({ ...prevSw, isEditingName: !prevSw.isEditingName }));
  };

  return (
    <div className="bg-gray-800 rounded-xl shadow-lg w-full overflow-hidden">
      <div className="flex items-center p-4">
        {/* Left side: Play/Pause button */}
        <button
          onClick={handleStartStop}
          className={`w-16 h-16 rounded-full flex items-center justify-center shadow-lg flex-shrink-0
            ${isRunning ? 'bg-red-600 hover:bg-red-700' : 'bg-green-600 hover:bg-green-700'} text-white`}
          aria-label={isRunning ? "Detener cronómetro" : "Iniciar cronómetro"}
        >
          {isRunning ? <PauseIcon size={32} /> : <PlayIcon size={32} />}
        </button>

        {/* Middle: Name and Times */}
        <div className="flex-1 ml-4 overflow-hidden">
          <div className="flex items-center mb-1">
            {isEditingName ? (
              <input
                type="text"
                ref={nameInputRef}
                value={name}
                onChange={handleNameChange}
                onBlur={handleToggleEditName}
                className="bg-gray-700 text-white text-lg font-semibold rounded p-1 w-full focus:outline-none focus:ring-1 focus:ring-blue-500"
                aria-label={`Nombre del cronómetro ${id + 1}`}
              />
            ) : (
              <span className="text-lg font-semibold text-white truncate">{name}</span>
            )}
            <button
              onClick={handleToggleEditName}
              className="ml-2 text-gray-400 hover:text-white flex-shrink-0"
              aria-label="Editar nombre"
            >
              <Edit2Icon size={16} />
            </button>
          </div>
          <div className="text-xl font-mono text-white">{formatTime(elapsedTime)}</div>
          <div className="text-md font-mono text-gray-400">{formatTime(lapTime)}</div>
        </div>

        {/* Right side: Lap, Reset, and Expand/Collapse arrow */}
        <div className="flex items-center space-x-2 ml-4 flex-shrink-0">
          <button
            onClick={handleLap}
            disabled={!isRunning}
            className={`w-10 h-10 rounded-full flex items-center justify-center transition duration-200 ease-in-out
              ${isRunning
                ? 'bg-blue-600 hover:bg-blue-700 text-white shadow-md'
                : 'bg-gray-700 text-gray-500 cursor-not-allowed'
              }`}
            aria-label="Registrar vuelta"
          >
            <PlusIcon size={20} />
          </button>
          <button
            onClick={handleReset}
            className="w-10 h-10 rounded-full flex items-center justify-center bg-gray-600 hover:bg-gray-700 text-white shadow-md transition duration-200 ease-in-out"
            aria-label="Reiniciar cronómetro"
          >
            <RotateCcwIcon size={20} />
          </button>
          <button
            onClick={() => setIsExpanded(!isExpanded)}
            className="text-gray-400 hover:text-white transition-transform duration-300"
            aria-label={isExpanded ? "Contraer historial de vueltas" : "Expandir historial de vueltas"}
          >
            <ChevronRightIcon size={24} className={isExpanded ? 'rotate-90' : ''} />
          </button>
        </div>
      </div>

      {isExpanded && laps.length > 0 && (
        <div className="p-4 pt-0 w-full">
          <div className="w-full max-h-40 overflow-y-auto bg-gray-900 rounded-lg p-2 border border-gray-700">
            <h3 className="text-md font-semibold text-gray-300 mb-2 text-center">Vueltas</h3>
            <ul className="space-y-2">
              {laps.slice().reverse().map((lap, idx) => {
                // Calculate duration of this specific lap
                const previousLapTime = idx < laps.length - 1 ? laps[laps.length - 1 - (idx + 1)].time : 0;
                const lapDuration = lap.time - previousLapTime;

                return (
                  <li key={lap.id} className="flex justify-between items-center text-white bg-gray-800 p-3 rounded-md">
                    <span className="text-lg text-gray-400">🏁 {laps.length - idx}</span>
                    <div className="flex flex-col items-end">
                      <span className="text-xl font-mono">{formatTime(lap.time)}</span>
                      <span className="text-sm text-gray-500">+{formatTime(lapDuration)}</span>
                    </div>
                  </li>
                );
              })}
            </ul>
          </div>
        </div>
      )}
    </div>
  );
};

const App = () => {
  const [stopwatches, setStopwatches] = useState([]);

  // Initialize stopwatches
  useEffect(() => {
    const initialStopwatches = Array.from({ length: 20 }, (_, i) => ({
      id: i,
      name: `Cronómetro ${i + 1}`,
      isRunning: false,
      elapsedTime: 0,
      lapTime: 0,
      laps: [],
      startTimeRef: { current: 0 }, // Use plain object for refs in state
      lapStartTimeRef: { current: 0 }, // Use plain object for refs in state
      intervalRef: { current: null }, // Use plain object for refs in state
      isEditingName: false,
    }));
    setStopwatches(initialStopwatches);
  }, []);

  // Function to update a specific stopwatch's state
  const updateStopwatchState = (id, updater) => {
    setStopwatches(prevStopwatches =>
      prevStopwatches.map(sw => (sw.id === id ? updater(sw) : sw))
    );
  };

  // Effect for managing stopwatch intervals
  useEffect(() => {
    stopwatches.forEach((sw) => {
      // Clear any existing interval to prevent duplicates
      clearInterval(sw.intervalRef.current);

      if (sw.isRunning) {
        if (sw.startTimeRef.current === 0) {
          sw.startTimeRef.current = Date.now() - sw.elapsedTime;
          sw.lapStartTimeRef.current = Date.now() - sw.lapTime;
        }

        sw.intervalRef.current = setInterval(() => {
          updateStopwatchState(sw.id, (prevSw) => {
            const now = Date.now();
            return {
              ...prevSw,
              elapsedTime: now - prevSw.startTimeRef.current,
              lapTime: now - prevSw.lapStartTimeRef.current,
            };
          });
        }, 10);
      }
    });

    // Cleanup function to clear all intervals when component unmounts or stopwatches change
    return () => {
      stopwatches.forEach(sw => clearInterval(sw.intervalRef.current));
    };
  }, [stopwatches]); // Dependency array: re-run if stopwatches array structure changes

  return (
    <div className="min-h-screen bg-gray-900 p-4 font-sans flex flex-col items-center">
      <script src="https://cdn.tailwindcss.com"></script>
      <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
      <style>
        {`
          body {
            font-family: 'Inter', sans-serif;
          }
        `}
      </style>

      {/* Main list view */}
      <div className="w-full max-w-md bg-gray-800 rounded-xl shadow-2xl overflow-hidden">
        <div className="flex items-center p-4 border-b border-gray-700">
          <button className="text-gray-400 hover:text-white mr-4" aria-label="Cerrar">
            <XIcon size={24} />
          </button>
          <h1 className="text-xl font-semibold text-white flex-1">Lista de cronómetro</h1>
        </div>
        <div className="divide-y divide-gray-700">
          {stopwatches.map((stopwatch) => (
            <StopwatchItem
              key={stopwatch.id}
              stopwatch={stopwatch}
              updateStopwatchState={updateStopwatchState}
            />
          ))}
        </div>
      </div>
    </div>
  );
};

export default App;

