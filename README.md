<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Задачи с подзадачами - Liquid Glass</title>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100vh;
      padding: 20px;
      color: white;
    }
    .app { max-width: 800px; margin: 0 auto; }
    h1, h2, h3 { text-shadow: 0 2px 10px rgba(0,0,0,0.3); }
    .glass-card {
      background: rgba(255, 255, 255, 0.1);
      backdrop-filter: blur(20px);
      border: 1px solid rgba(255, 255, 255, 0.2);
      border-radius: 20px;
      padding: 20px;
      margin: 20px 0;
      box-shadow: 0 8px 32px rgba(0,0,0,0.3);
      transition: transform 0.3s;
    }
    .glass-card:hover { transform: translateY(-5px); }
    input, button {
      background: rgba(255, 255, 255, 0.2);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(255, 255, 255, 0.3);
      border-radius: 10px;
      padding: 12px;
      margin: 5px;
      color: white;
      font-size: 16px;
    }
    button { cursor: pointer; background: rgba(255, 255, 255, 0.3); }
    button:hover { background: rgba(255, 255, 255, 0.4); }
    button:disabled { opacity: 0.5; cursor: not-allowed; }
    ul { list-style: none; }
    li { display: flex; align-items: center; margin: 10px 0; }
    input[type="checkbox"] { width: 20px; height: 20px; margin-right: 10px; accent-color: #fff; }
    .task { animation: slideIn 0.5s; }
    @keyframes slideIn { from { opacity: 0; transform: translateX(-20px); } to { opacity: 1; transform: translateX(0); } }
    @media (max-width: 600px) { .glass-card { padding: 15px; } }
  </style>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      const [tasks, setTasks] = useState([]);
      const [currentTask, setCurrentTask] = useState('');
      const [history, setHistory] = useState([]);

      useEffect(() => {
        const savedTasks = localStorage.getItem('tasks');
        if (savedTasks) setTasks(JSON.parse(savedTasks));
        const savedHistory = localStorage.getItem('history');
        if (savedHistory) setHistory(JSON.parse(savedHistory));
      }, []);

      useEffect(() => {
        localStorage.setItem('tasks', JSON.stringify(tasks));
        localStorage.setItem('history', JSON.stringify(history));
      }, [tasks, history]);

      const addTask = () => {
        if (currentTask.trim()) {
          const newTask = {
            id: Date.now(),
            name: currentTask.trim(),
            subtasks: [],
            completed: false
          };
          setTasks([...tasks, newTask]);
          setCurrentTask('');
        }
      };

      const addSubtask = (taskId) => {
        const newSubName = prompt('Подзадача (напр. "нарезать помидоры"):');
        if (newSubName && newSubName.trim()) {
          setTasks(tasks.map(task =>
            task.id === taskId
              ? {
                  ...task,
                  subtasks: [...task.subtasks, {
                    id: Date.now() + Math.random(),
                    name: newSubName.trim(),
                    done: false
                  }]
                }
              : task
          ));
        }
      };

      const toggleSubtask = (taskId, subId) => {
        setTasks(tasks.map(task => {
          if (task.id === taskId) {
            const updatedSubtasks = task.subtasks.map(sub =>
              sub.id === subId ? { ...sub, done: !sub.done } : sub
            );
            return {
              ...task,
              subtasks: updatedSubtasks,
              completed: updatedSubtasks.every(s => s.done)
            };
          }
          return task;
        }));
      };

      const completeTask = (taskId) => {
        const task = tasks.find(t => t.id === taskId);
        if (task && task.completed) {
          setHistory([task, ...history]);
          setTasks(tasks.filter(t => t.id !== taskId));
        }
      };

      const deleteTask = (taskId) => {
        setTasks(tasks.filter(t => t.id !== taskId));
      };

      return (
        <div className="app">
          <h1>Задачи с подзадачами (Liquid Glass)</h1>
          
          <div className="glass-card">
            <input
              value={currentTask}
              onChange={e => setCurrentTask(e.target.value)}
              placeholder="Новая задача (напр. 'диетический салат')"
              onKeyPress={e => e.key === 'Enter' && addTask()}
            />
            <button onClick={addTask}>Добавить задачу</button>
          </div>

          <section>
            <h2>Активные задачи</h2>
            {tasks.length === 0 ? (
              <p style={{textAlign: 'center', opacity: 0.7}}>Нет задач. Добавьте первую!</p>
            ) : (
              tasks.map(task => (
                <div key={task.id} className="glass-card task">
                  <div style={{display: 'flex', justifyContent: 'space-between', alignItems: 'center'}}>
                    <h3>{task.name} {task.completed && '(завершена)'}</h3>
                    <div>
                      <button onClick={() => addSubtask(task.id)}>Подзадача</button>
                      {!task.completed && (
                        <button onClick={() => deleteTask(task.id)}>Удалить</button>
                      )}
                    </div>
                  </div>
                  <ul>
                    {task.subtasks.map(sub => (
                      <li key={sub.id}>
                        <input
                          type="checkbox"
                          checked={sub.done}
                          onChange={() => toggleSubtask(task.id, sub.id)}
                        />
                        <span style={{textDecoration: sub.done ? 'line-through' : 'none', opacity: sub.done ? 0.7 : 1}}>
                          {sub.name}
                        </span>
                      </li>
                    ))}
                  </ul>
                  {task.completed && (
                    <button onClick={() => completeTask(task.id)}>В историю</button>
                  )}
                </div>
              ))
            )}
          </section>

          <section>
            <h2>История (последние 10)</h2>
            {history.length === 0 ? (
              <p style={{textAlign: 'center', opacity: 0.7}}>История пуста</p>
            ) : (
              history.slice(0, 10).map(task => (
                <div key={task.id} className="glass-card" style={{opacity: 0.8}}>
                  ✅ {task.name} ({task.subtasks.length} подзадач)
                </div>
              ))
            )}
          </section>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
