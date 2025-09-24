import React, { useRef, useState, useEffect } from 'react';

// --- DATABASE HELPERS (using localStorage for offline support) ---
const DB = {
  get: () => {
    const db = localStorage.getItem('aiFitnessCoachDB');
    // Add customExercises array if it doesn't exist
    const defaults = { users: [], history: {}, customExercises: [] };
    const data = db ? JSON.parse(db) : defaults;
    return { ...defaults, ...data };
  },
  set: (db) => {
    localStorage.setItem('aiFitnessCoachDB', JSON.stringify(db));
  }
};

// --- AI & LOGIC HELPERS ---

const calculateBmi = (weight, height) => {
  if (!weight || !height) return { bmi: null, category: 'N/A' };
  const heightInMeters = height / 100;
  const bmi = (weight / (heightInMeters * heightInMeters)).toFixed(1);
  let category = 'Normal';
  if (bmi < 18.5) category = 'Underweight';
  else if (bmi >= 25 && bmi < 29.9) category = 'Overweight';
  else if (bmi >= 30) category = 'Obesity';
  return { bmi, category };
};

// **ENHANCED AI PLAN GENERATOR**
const generateAIPlan = (user) => {
  const { category } = calculateBmi(user.weight, user.height);
  
  const plans = {
    'Build Muscle': {
      'Underweight': [
        { day: "Day 1: Upper Body", exercises: [{ name: 'Push-ups', sets: 4, reps: 8 }, { name: 'Overhead Press', sets: 3, reps: 8 }] },
        { day: "Day 2: Lower Body", exercises: [{ name: 'Squats', sets: 4, reps: 10 }] },
        { day: "Day 3: Upper Body", exercises: [{ name: 'Push-ups', sets: 4, reps: 8 }, { name: 'Bicep Curls', sets: 4, reps: 10 }] },
      ],
      'Default': [
        { day: "Day 1: Full Body", exercises: [{ name: 'Squats', sets: 3, reps: 12 }, { name: 'Push-ups', sets: 3, reps: 12 }, { name: 'Bicep Curls', sets: 3, reps: 12 }] },
        { day: "Day 2: Rest", exercises: [] },
        { day: "Day 3: Full Body", exercises: [{ name: 'Squats', sets: 3, reps: 12 }, { name: 'Overhead Press', sets: 3, reps: 10 }, { name: 'Bicep Curls', sets: 3, reps: 12 }] },
      ]
    },
    'Lose Weight': {
      'Overweight': [
         { day: "Day 1: Full Body Strength", exercises: [{ name: 'Squats', sets: 3, reps: 15 }, { name: 'Push-ups', sets: 3, reps: 15 }] },
         { day: "Day 2: Rest", exercises: [] },
         { day: "Day 3: Full Body Strength", exercises: [{ name: 'Squats', sets: 3, reps: 15 }, { name: 'Bicep Curls', sets: 3, reps: 15 }] },
      ],
      'Default': [
        { day: "Day 1: Full Body Circuit", exercises: [{ name: 'Squats', sets: 2, reps: 15 }, { name: 'Push-ups', sets: 2, reps: 12 }] },
        { day: "Day 2: Rest", exercises: [] },
        { day: "Day 3: Full Body Circuit", exercises: [{ name: 'Squats', sets: 2, reps: 15 }, { name: 'Bicep Curls', sets: 2, reps: 15 }] },
      ]
    }
  };

  const goalPlan = plans[user.goal];
  return goalPlan[category] || goalPlan['Default'];
};

const TRACKABLE_EXERCISES = ['Bicep Curls', 'Push-ups', 'Squats', 'Overhead Press'];


// --- STYLED COMPONENTS ---
const Button = ({ onClick, children, style, disabled }) => ( <button onClick={onClick} style={{...styles.button, ...(disabled ? styles.buttonDisabled : {}), ...style}} disabled={disabled}>{children}</button> );
const Card = ({ children, style }) => ( <div style={{...styles.card, ...style}}>{children}</div> );

// --- MAIN APP COMPONENTS ---

function WelcomeScreen({ onGetStarted }) {
    return (
        <div style={styles.welcomeContainer}>
            <h1 style={styles.mainTitle}>Nexus Fit</h1>
            <p style={styles.quote}>"The only bad workout is the one that didn't happen."</p>
            <Button onClick={onGetStarted} style={{marginTop: '20px'}}>Get Started</Button>
        </div>
    );
}

function LiveWorkout({ exercise, onFinishWorkout }) {
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const animationFrameId = useRef(null);
  const stageRef = useRef(exercise === 'Push-ups' || exercise === 'Squats' ? 'up' : 'down');
  const [counter, setCounter] = useState(0);
  const [feedback, setFeedback] = useState('Initializing...');

  useEffect(() => {
    const loadScript = (src) => new Promise((resolve, reject) => {
        const script = document.createElement('script'); script.src = src; script.async = true;
        script.onload = resolve; script.onerror = () => reject(new Error(`Failed to load script ${src}`));
        document.head.appendChild(script);
    });
    const initialize = async () => {
        setFeedback('Loading AI libraries...');
        try {
            if (!window.tf) await loadScript('https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.11.0/dist/tf.min.js');
            if (!window.poseDetection) await loadScript('https://cdn.jsdelivr.net/npm/@tensorflow-models/pose-detection@2.0.0/dist/pose-detection.min.js');
            setFeedback('AI Libraries loaded. Accessing camera...');
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            if (videoRef.current) {
                videoRef.current.srcObject = stream;
                videoRef.current.onloadedmetadata = async () => {
                    await videoRef.current.play();
                    const detector = await window.poseDetection.createDetector(window.poseDetection.SupportedModels.MoveNet, { modelType: window.poseDetection.movenet.modelType.SINGLEPOSE_LIGHTNING });
                    setFeedback(""); runPoseDetection(detector);
                };
            }
        } catch (error) { setFeedback(`Error: ${error.message}`); }
    };
    initialize();
    return () => { if (animationFrameId.current) cancelAnimationFrame(animationFrameId.current); };
  }, [exercise]);

  const runPoseDetection = (detector) => {
    const detectLoop = () => { detect(detector); animationFrameId.current = requestAnimationFrame(detectLoop); };
    detectLoop();
  };
  const detect = async (detector) => {
    if (videoRef.current && videoRef.current.readyState === 4) {
      const video = videoRef.current;
      const poses = await detector.estimatePoses(video);
      const ctx = canvasRef.current.getContext('2d');
      drawCanvas(poses, video.videoWidth, video.videoHeight, ctx);
      if (poses.length > 0) {
        const keypoints = poses[0].keypoints;
        const shoulder = keypoints.find(k=>k.name==='left_shoulder'), elbow = keypoints.find(k=>k.name==='left_elbow'), wrist = keypoints.find(k=>k.name==='left_wrist'), hip = keypoints.find(k=>k.name==='left_hip'), knee = keypoints.find(k=>k.name==='left_knee'), ankle = keypoints.find(k=>k.name==='left_ankle');

        if (exercise === 'Bicep Curls') {
            if (shoulder && elbow && wrist && shoulder.score > 0.3 && elbow.score > 0.3 && wrist.score > 0.3) {
                const angle = calculateAngle(shoulder, elbow, wrist);
                if (angle > 160) { if (stageRef.current !== 'down') stageRef.current = 'down'; }
                if (angle < 40 && stageRef.current === 'down') { stageRef.current = 'up'; setCounter(prev => prev + 1); }
            }
        } else if (exercise === 'Push-ups') {
            if (shoulder && elbow && wrist && hip && ankle) {
                const elbowAngle = calculateAngle(shoulder, elbow, wrist), backAngle = calculateAngle(shoulder, hip, ankle);
                if (backAngle < 160) setFeedback("Keep your back straight!"); else setFeedback("");
                if (elbowAngle < 90) { if (stageRef.current !== 'down') stageRef.current = 'down'; }
                if (elbowAngle > 160 && stageRef.current === 'down') { stageRef.current = 'up'; setCounter(prev => prev + 1); setFeedback("Great Push-up!"); }
            }
        } else if (exercise === 'Squats') {
            if (hip && knee && ankle) {
                const kneeAngle = calculateAngle(hip, knee, ankle);
                if (kneeAngle < 90) { if(stageRef.current !== 'down') setFeedback("Good depth!"); stageRef.current = 'down'; }
                if (kneeAngle > 160 && stageRef.current === 'down') { stageRef.current = 'up'; setCounter(prev => prev + 1); setFeedback(""); }
            }
        } else if (exercise === 'Overhead Press') {
            if (shoulder && elbow && wrist) {
                const elbowAngle = calculateAngle(shoulder, elbow, wrist);
                if (elbowAngle > 160 && wrist.y < shoulder.y) { if (stageRef.current !== 'up') { setCounter(prev => prev + 1); } stageRef.current = 'up'; }
                if (elbowAngle < 90 && wrist.y >= shoulder.y) { stageRef.current = 'down'; }
            }
        }
      }
    }
  };
  const drawCanvas = (poses, w, h, ctx) => {
    if (canvasRef.current) {
        canvasRef.current.width = w; canvasRef.current.height = h;
        if (poses.length > 0) { drawSkeleton(poses[0].keypoints, ctx); drawKeypoints(poses[0].keypoints, ctx); }
    }
  };
  return (
    <div>
      <h1 style={styles.title}>Live Workout: {exercise}</h1>
      <div style={styles.webcamContainer}><video ref={videoRef} autoPlay playsInline muted style={styles.webcam} /><canvas ref={canvasRef} style={styles.canvas} /><div style={styles.statusBox}><div style={styles.metric}><span style={styles.metricLabel}>REPS</span><span style={styles.metricValue}>{counter}</span></div></div>{feedback && <div style={styles.feedbackBox}>{feedback}</div>}</div>
      <Button onClick={() => onFinishWorkout(counter)}>Finish Workout</Button>
    </div>
  );
}

function Dashboard({ user, onStartWorkout, onSwitchUser, onEditPlan }) {
  const plan = user.customPlan || generateAIPlan(user);
  const { bmi, category } = calculateBmi(user.weight, user.height);
  return (
    <div>
      <h1 style={styles.title}>Welcome back, {user.name}!</h1>
      <div style={styles.gamificationGrid}>
        <Card style={styles.statCard}><span style={styles.metricLabel}>Points</span><span style={styles.metricValue}>{user.stats.points} ✨</span></Card>
        <Card style={styles.statCard}><span style={styles.metricLabel}>Workout Streak</span><span style={styles.metricValue}>{user.stats.streak} 🔥</span></Card>
        {bmi && <Card style={styles.statCard}><span style={styles.metricLabel}>BMI</span><span style={styles.metricValue}>{bmi}</span><span style={{fontSize: '14px', color: '#ccc'}}>{category}</span></Card>}
      </div>
      <Card style={{marginTop: '30px'}}>
        <div style={styles.planHeader}>
          <h2 style={styles.cardTitle}>{user.customPlan ? "Your Custom Plan" : "AI Suggested Plan"}</h2>
          <Button onClick={onEditPlan} style={{fontSize: '14px', padding: '8px 15px'}}>{user.customPlan ? "Edit Plan" : "Customize"}</Button>
        </div>
        {plan.map(day => (
          <div key={day.day} style={styles.dayContainer}>
            <h3 style={styles.dayTitle}>{day.day}</h3>
            {day.exercises.length === 0 ? <p style={{color: '#888'}}>Rest Day</p> : day.exercises.map((item, index) => (
              <div key={item.name + index} style={styles.planItem}>
                <span>{item.name} ({item.sets ? `${item.sets} sets, ${item.reps} reps` : item.duration})</span>
                <Button onClick={() => onStartWorkout(item.name)} disabled={!TRACKABLE_EXERCISES.includes(item.name)}>{TRACKABLE_EXERCISES.includes(item.name) ? 'Start' : 'Manual'}</Button>
              </div>
            ))}
          </div>
        ))}
      </Card>
      <Button onClick={onSwitchUser} style={{marginTop: '40px', backgroundColor: '#6c757d'}}>Switch User</Button>
    </div>
  );
}

function PlanEditor({ user, customExercises, onSavePlan, onBack, onAddCustomExercise }) {
  const [customPlan, setCustomPlan] = useState(user.customPlan || generateAIPlan(user));
  const [newExerciseName, setNewExerciseName] = useState("");

  const availableExercises = [...TRACKABLE_EXERCISES, ...customExercises];

  const handleExerciseChange = (dayIndex, exIndex, field, value) => {
    const newPlan = [...customPlan];
    newPlan[dayIndex].exercises[exIndex][field] = value;
    setCustomPlan(newPlan);
  };
  
  const addExercise = (dayIndex) => {
    const newPlan = [...customPlan];
    newPlan[dayIndex].exercises.push({ name: 'Bicep Curls', sets: 3, reps: 10 });
    setCustomPlan(newPlan);
  };

  const removeExercise = (dayIndex, exIndex) => {
    const newPlan = [...customPlan];
    newPlan[dayIndex].exercises.splice(exIndex, 1);
    setCustomPlan(newPlan);
  };

  const handleAddNewExercise = () => {
    if (newExerciseName && !availableExercises.includes(newExerciseName)) {
      onAddCustomExercise(newExerciseName);
      setNewExerciseName("");
    }
  };

  return (
    <Card>
      <h2 style={styles.cardTitle}>Edit Your Workout Plan</h2>
      {customPlan.map((day, dayIndex) => (
        <div key={day.day} style={styles.dayContainer}>
          <h3 style={styles.dayTitle}>{day.day}</h3>
          {day.exercises.map((ex, exIndex) => (
            <div style={styles.editorItem} key={exIndex}>
              <select value={ex.name} onChange={(e) => handleExerciseChange(dayIndex, exIndex, 'name', e.target.value)} style={{...styles.input, flex: 2}}>
                {availableExercises.map(name => <option key={name}>{name}</option>)}
              </select>
              <input type="number" value={ex.sets} onChange={(e) => handleExerciseChange(dayIndex, exIndex, 'sets', parseInt(e.target.value))} placeholder="Sets" style={{...styles.input, flex: 1}} />
              <input type="number" value={ex.reps} onChange={(e) => handleExerciseChange(dayIndex, exIndex, 'reps', parseInt(e.target.value))} placeholder="Reps" style={{...styles.input, flex: 1}} />
              <Button onClick={() => removeExercise(dayIndex, exIndex)} style={{background: '#dc3545', padding: '10px 15px'}}>X</Button>
            </div>
          ))}
          <Button onClick={() => addExercise(dayIndex)} style={{marginTop: '10px', fontSize: '16px'}}>+ Add Exercise</Button>
        </div>
      ))}
      <div style={styles.dayContainer}>
          <h3 style={styles.dayTitle}>Create New Exercise</h3>
          <div style={styles.editorItem}>
              <input type="text" value={newExerciseName} onChange={(e) => setNewExerciseName(e.target.value)} placeholder="E.g., Dumbbell Rows" style={{...styles.input, flex: 2}} />
              <Button onClick={handleAddNewExercise} style={{flex: 1}}>Add to List</Button>
          </div>
      </div>
      <div style={{display: 'flex', justifyContent: 'space-between', marginTop: '30px'}}>
        <Button onClick={onBack} style={{background: '#6c757d'}}>Back</Button>
        <Button onClick={() => onSavePlan(customPlan)}>Save Plan</Button>
      </div>
    </Card>
  );
}

function UserSelection({ users, onSelectUser, onAddUser }) {
  const [newName, setNewName] = useState(''), [newGoal, setNewGoal] = useState('Build Muscle'), [newWeight, setNewWeight] = useState(''), [newHeight, setNewHeight] = useState('');
  const handleAddUser = () => {
    if (newName.trim() && newWeight && newHeight) {
      onAddUser(newName.trim(), newGoal, parseFloat(newWeight), parseFloat(newHeight));
      setNewName(''); setNewWeight(''); setNewHeight('');
    }
  };
  return (
    <div>
      <h1 style={styles.title}>Who's Working Out?</h1>
      <div style={styles.userList}>{users.map(user => (<div key={user.id} style={styles.userItem} onClick={() => onSelectUser(user)}><div style={styles.userAvatar}>{user.name.charAt(0)}</div><span>{user.name}</span></div>))}</div>
      <Card style={{marginTop: '40px'}}>
        <h2 style={styles.cardTitle}>Add New User</h2>
        <input type="text" value={newName} onChange={e => setNewName(e.target.value)} placeholder="Enter name" style={styles.input} />
        <input type="number" value={newWeight} onChange={e => setNewWeight(e.target.value)} placeholder="Weight (kg)" style={styles.input} />
        <input type="number" value={newHeight} onChange={e => setNewHeight(e.target.value)} placeholder="Height (cm)" style={styles.input} />
        <select value={newGoal} onChange={e => setNewGoal(e.target.value)} style={styles.input}><option>Build Muscle</option><option>Lose Weight</option></select>
        <Button onClick={handleAddUser}>Add User</Button>
      </Card>
    </div>
  );
}

// --- ROOT APP COMPONENT ---
function App() {
  const [dbState, setDbState] = useState(DB.get()), [currentUser, setCurrentUser] = useState(null), [currentScreen, setCurrentScreen] = useState('welcome'), [currentExercise, setCurrentExercise] = useState(null);
  const updateUser = (updatedUser) => { const newDb = { ...dbState, users: dbState.users.map(u => u.id === updatedUser.id ? updatedUser : u) }; DB.set(newDb); setDbState(newDb); setCurrentUser(updatedUser); };
  const handleSelectUser = (user) => { setCurrentUser(user); setCurrentScreen('dashboard'); };
  const handleAddUser = (name, goal, weight, height) => {
    const newUser = { id: `user_${Date.now()}`, name, goal, weight, height, stats: { points: 0, streak: 0 }, customPlan: null };
    const newDb = { ...dbState, users: [...dbState.users, newUser] }; DB.set(newDb); setDbState(newDb);
  };
  const handleStartWorkout = (exerciseName) => { setCurrentExercise(exerciseName); setCurrentScreen('workout'); };
  const handleFinishWorkout = (repsCompleted) => {
    const pointsEarned = repsCompleted * 5;
    const updatedUser = { ...currentUser, stats: { ...currentUser.stats, points: currentUser.stats.points + pointsEarned, streak: currentUser.stats.streak + 1 } };
    updateUser(updatedUser); setCurrentScreen('dashboard');
  };
  const handleSavePlan = (plan) => {
    const updatedUser = { ...currentUser, customPlan: plan };
    updateUser(updatedUser); setCurrentScreen('dashboard');
  };
  const handleAddCustomExercise = (exerciseName) => {
      const newDb = { ...dbState, customExercises: [...dbState.customExercises, exerciseName] };
      DB.set(newDb);
      setDbState(newDb);
  };

  return (
    <div style={styles.app}>
      <style>{` 
        body { 
          margin: 0; 
          padding: 0; 
          font-family: sans-serif; 
          background: linear-gradient(rgba(0, 0, 0, 0.8), rgba(0, 0, 0, 0.8)), url('https://images.unsplash.com/photo-1571902943202-507ec2618e8f?q=80&w=2575&auto=format&fit=crop');
          background-size: cover;
          background-position: center;
          background-attachment: fixed;
        } 
      `}</style>
      <header style={styles.appHeader}>
        {currentScreen === 'welcome' && <WelcomeScreen onGetStarted={() => setCurrentScreen('users')} />}
        {currentScreen === 'users' && <UserSelection users={dbState.users} onSelectUser={handleSelectUser} onAddUser={handleAddUser} />}
        {currentScreen === 'dashboard' && <Dashboard user={currentUser} onStartWorkout={handleStartWorkout} onSwitchUser={() => setCurrentScreen('users')} onEditPlan={() => setCurrentScreen('planEditor')} />}
        {currentScreen === 'planEditor' && <PlanEditor user={currentUser} customExercises={dbState.customExercises} onSavePlan={handleSavePlan} onBack={() => setCurrentScreen('dashboard')} onAddCustomExercise={handleAddCustomExercise}/>}
        {currentScreen === 'workout' && <LiveWorkout user={currentUser} exercise={currentExercise} onFinishWorkout={handleFinishWorkout} />}
      </header>
    </div>
  );
}

// --- Drawing Utilities ---
function calculateAngle(a,b,c){if(!a||!b||!c)return null;const rads=Math.atan2(c.y-b.y,c.x-b.x)-Math.atan2(a.y-b.y,a.x-b.x);let ang=Math.abs(rads*180.0/Math.PI);if(ang>180.0)ang=360-ang;return ang;}
function drawKeypoints(keypoints,ctx){for(let kp of keypoints){if(kp.score>0.3){ctx.beginPath();ctx.arc(kp.x,kp.y,6,0,2*Math.PI);ctx.fillStyle='#4CAF50';ctx.fill();}}}
function drawSkeleton(keypoints,ctx){const c=[['left_shoulder','right_shoulder'],['left_shoulder','left_elbow'],['right_shoulder','right_elbow'],['left_elbow','left_wrist'],['right_elbow','right_wrist'],['left_shoulder','left_hip'],['right_shoulder','right_hip'],['left_hip','right_hip'],['left_hip','left_knee'],['right_hip','right_knee'],['left_knee','left_ankle'],['right_knee','right_ankle']];ctx.strokeStyle='#FFFFFF';ctx.lineWidth=3;const m=new Map(keypoints.map(kp=>[kp.name,kp]));for(const[s,e]of c){const p1=m.get(s),p2=m.get(e);if(p1&&p2&&p1.score>0.3&&p2.score>0.3){ctx.beginPath();ctx.moveTo(p1.x,p1.y);ctx.lineTo(p2.x,p2.y);ctx.stroke();}}}

// --- STYLES ---
const styles = {
    app: { textAlign: 'center' }, appHeader: { minHeight: '100vh', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center', color: 'white', padding: '20px' },
    title: { fontSize: 'calc(20px + 2vmin)', marginBottom: '30px', textShadow: '0 3px 6px rgba(0,0,0,0.4)' },
    mainTitle: { fontSize: 'calc(40px + 4vmin)', textShadow: '0 5px 15px rgba(0,0,0,0.5)', margin: 0 },
    quote: { fontSize: 'calc(12px + 1vmin)', fontStyle: 'italic', color: '#ccc', marginTop: '10px' },
    welcomeContainer: { display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center' },
    webcamContainer: { position: 'relative', width: 'clamp(320px, 70vw, 800px)', paddingBottom: 'calc(clamp(320px, 70vw, 800px) * 0.75)', borderRadius: '15px', overflow: 'hidden', boxShadow: '0 10px 20px rgba(0,0,0,0.19), 0 6px 6px rgba(0,0,0,0.23)' },
    webcam: { position: 'absolute', top: 0, left: 0, width: '100%', height: '100%', objectFit: 'cover', transform: 'scaleX(-1)' },
    canvas: { position: 'absolute', top: 0, left: 0, width: '100%', height: '100%', zIndex: 1, transform: 'scaleX(-1)' },
    statusBox: { position: 'absolute', top: '20px', left: '20px', backgroundColor: 'rgba(0,0,0,0.7)', padding: '15px', borderRadius: '10px', zIndex: 2 },
    metric: { display: 'flex', flexDirection: 'column', alignItems: 'center' }, metricLabel: { fontSize: '16px', color: '#aaa', marginBottom: '5px' },
    metricValue: { fontSize: '36px', fontWeight: 'bold', color: '#4CAF50' }, feedbackBox: { position: 'absolute', bottom: '20px', left: '50%', transform: 'translateX(-50%)', backgroundColor: 'rgba(0,0,0,0.7)', color: 'white', padding: '10px 20px', borderRadius: '8px', zIndex: 2 },
    button: { background: '#4CAF50', color: 'white', border: 'none', padding: '12px 25px', borderRadius: '8px', fontSize: '18px', cursor: 'pointer', transition: 'background 0.2s' },
    buttonDisabled: { background: '#555', cursor: 'not-allowed' }, 
    card: { background: 'rgba(28, 38, 54, 0.8)', padding: '20px', borderRadius: '15px', width: 'clamp(320px, 80vw, 700px)', backdropFilter: 'blur(10px)', border: '1px solid rgba(255, 255, 255, 0.1)' },
    statCard: {
        background: 'rgba(28, 38, 54, 0.8)', padding: '20px', borderRadius: '15px', backdropFilter: 'blur(10px)', border: '1px solid rgba(255, 255, 255, 0.1)',
        display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center',
        width: 'auto' // Fix to allow grid to size the cards
    },
    cardTitle: { marginTop: 0, color: '#eee' }, input: { width: 'calc(100% - 24px)', padding: '12px', margin: '8px 0', borderRadius: '5px', border: '1px solid #555', background: '#333', color: 'white', fontSize: '16px' },
    userList: { display: 'flex', gap: '20px', justifyContent: 'center', flexWrap: 'wrap' },
    userItem: { cursor: 'pointer', display: 'flex', flexDirection: 'column', alignItems: 'center', gap: '10px' },
    userAvatar: { width: '80px', height: '80px', borderRadius: '50%', background: '#4CAF50', display: 'flex', alignItems: 'center', justifyContent: 'center', fontSize: '36px', fontWeight: 'bold' },
    gamificationGrid: { display: 'grid', gridTemplateColumns: 'repeat(auto-fit, minmax(120px, 1fr))', gap: '20px', width: 'clamp(320px, 80vw, 700px)' },
    planHeader: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '20px' },
    dayContainer: { borderTop: '1px solid rgba(255,255,255,0.2)', paddingTop: '15px', marginTop: '15px' },
    dayTitle: { textAlign: 'left', color: '#4CAF50', margin: '0 0 10px 0' },
    planItem: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', padding: '10px 0' },
    editorItem: { display: 'flex', gap: '10px', alignItems: 'center', marginBottom: '10px' }
};

export default App;

