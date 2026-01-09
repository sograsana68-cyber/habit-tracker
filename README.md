import React, { useState, useEffect } from 'react';
import { Plus, Check, Trash2, TrendingUp, Calendar, Flame, Settings, HelpCircle, User, LogOut, Menu, X, Bell, Award, Target, Clock, Edit2, BarChart3, ChevronRight, Search, Lock, Mail, Eye, EyeOff } from 'lucide-react';

const HabitFlowApp = () => {
  const [currentUser, setCurrentUser] = useState(null);
  const [screen, setScreen] = useState('login');
  const [habits, setHabits] = useState([]);
  const [loading, setLoading] = useState(true);
  const [menuOpen, setMenuOpen] = useState(false);
  
  // Auth states
  const [authMode, setAuthMode] = useState('login');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [username, setUsername] = useState('');
  const [showPassword, setShowPassword] = useState(false);
  a
  // Habit states
  const [isAddingHabit, setIsAddingHabit] = useState(false);
  const [editingHabit, setEditingHabit] = useState(null);
  const [newHabit, setNewHabit] = useState({
    name: '',
    color: '#8b5cf6',
    category: 'health',
    goal: 'daily',
    reminder: false,
    reminderTime: '09:00',
    notes: ''
  });

  const colors = ['#8b5cf6', '#ec4899', '#f59e0b', '#10b981', '#3b82f6', '#ef4444', '#8b5cf6', '#06b6d4'];
  const categories = ['health', 'fitness', 'learning', 'work', 'mindfulness', 'social', 'creative', 'other'];
  const goalTypes = ['daily', 'weekly', 'custom'];

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const result = await window.storage.get('current-user', false);
      if (result && result.value) {
        const user = JSON.parse(result.value);
        setCurrentUser(user);
        loadUserHabits(user.id);
        setScreen('home');
      }
    } catch (error) {
      console.log('No user logged in');
    } finally {
      setLoading(false);
    }
  };

  const loadUserHabits = async (userId) => {
    try {
      const result = await window.storage.get(`habits-${userId}`, false);
      if (result && result.value) {
        setHabits(JSON.parse(result.value));
      }
    } catch (error) {
      console.log('No habits found');
    }
  };

  const saveHabits = async (updatedHabits) => {
    if (currentUser) {
      try {
        await window.storage.set(`habits-${currentUser.id}`, JSON.stringify(updatedHabits), false);
      } catch (error) {
        console.error('Failed to save habits:', error);
      }
    }
  };

  const handleAuth = async () => {
    if (authMode === 'login') {
      try {
        const result = await window.storage.get(`user-${email}`, false);
        if (result && result.value) {
          const user = JSON.parse(result.value);
          if (user.password === password) {
            setCurrentUser(user);
            await window.storage.set('current-user', JSON.stringify(user), false);
            loadUserHabits(user.id);
            setScreen('home');
          } else {
            alert('Invalid password');
          }
        } else {
          alert('User not found. Please sign up.');
        }
      } catch (error) {
        alert('User not found. Please sign up.');
      }
    } else {
      const newUser = {
        id: Date.now(),
        username,
        email,
        password,
        createdAt: new Date().toISOString()
      };
      try {
        await window.storage.set(`user-${email}`, JSON.stringify(newUser), false);
        await window.storage.set('current-user', JSON.stringify(newUser), false);
        setCurrentUser(newUser);
        setScreen('home');
      } catch (error) {
        alert('Failed to create account');
      }
    }
  };

  const handleLogout = async () => {
    try {
      await window.storage.delete('current-user', false);
      setCurrentUser(null);
      setHabits([]);
      setScreen('login');
      setMenuOpen(false);
    } catch (error) {
      console.error('Logout failed:', error);
    }
  };

  const addOrUpdateHabit = async () => {
    if (newHabit.name.trim()) {
      let updatedHabits;
      if (editingHabit) {
        updatedHabits = habits.map(h => 
          h.id === editingHabit.id 
            ? { ...editingHabit, ...newHabit, id: editingHabit.id, completedDates: editingHabit.completedDates }
            : h
        );
        setEditingHabit(null);
      } else {
        const habit = {
          ...newHabit,
          id: Date.now(),
          completedDates: [],
          createdAt: new Date().toISOString()
        };
        updatedHabits = [...habits, habit];
      }
      setHabits(updatedHabits);
      await saveHabits(updatedHabits);
      resetHabitForm();
    } else {
      alert('Please enter a habit name');
    }
  };

  const resetHabitForm = () => {
    setNewHabit({
      name: '',
      color: '#8b5cf6',
      category: 'health',
      goal: 'daily',
      reminder: false,
      reminderTime: '09:00',
      notes: ''
    });
    setIsAddingHabit(false);
    setEditingHabit(null);
  };

  const startEditHabit = (habit) => {
    setEditingHabit(habit);
    setNewHabit({
      name: habit.name,
      color: habit.color,
      category: habit.category,
      goal: habit.goal,
      reminder: habit.reminder,
      reminderTime: habit.reminderTime,
      notes: habit.notes || ''
    });
    setIsAddingHabit(true);
  };

  const toggleHabit = (habitId) => {
    const today = new Date().toDateString();
    const updatedHabits = habits.map(habit => {
      if (habit.id === habitId) {
        const isCompletedToday = habit.completedDates.includes(today);
        return {
          ...habit,
          completedDates: isCompletedToday
            ? habit.completedDates.filter(date => date !== today)
            : [...habit.completedDates, today]
        };
      }
      return habit;
    });
    setHabits(updatedHabits);
    saveHabits(updatedHabits);
  };

  const deleteHabit = (habitId) => {
    if (confirm('Are you sure you want to delete this habit?')) {
      const updatedHabits = habits.filter(habit => habit.id !== habitId);
      setHabits(updatedHabits);
      saveHabits(updatedHabits);
    }
  };

  const getStreak = (completedDates) => {
    if (completedDates.length === 0) return 0;
    const sortedDates = completedDates.map(d => new Date(d)).sort((a, b) => b - a);
    let streak = 0;
    let currentDate = new Date();
    currentDate.setHours(0, 0, 0, 0);
    
    for (let date of sortedDates) {
      date.setHours(0, 0, 0, 0);
      const diffDays = Math.floor((currentDate - date) / (1000 * 60 * 60 * 24));
      if (diffDays === streak) {
        streak++;
      } else if (diffDays > streak) {
        break;
      }
    }
    return streak;
  };

  const getCategoryIcon = (category) => {
    const icons = {
      health: '🏥',
      fitness: '💪',
      learning: '📚',
      work: '💼',
      mindfulness: '🧘',
      social: '👥',
      creative: '🎨',
      other: '⭐'
    };
    return icons[category] || '⭐';
  };

  const today = new Date().toDateString();
  const totalCompletedToday = habits.filter(h => h.completedDates.includes(today)).length;
  const bestStreak = Math.max(0, ...habits.map(h => getStreak(h.completedDates)));

  // Login/Signup Screen
  if (loading) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-blue-500 flex items-center justify-center">
        <div className="text-3xl text-white animate-pulse font-bold">HabitFlow</div>
      </div>
    );
  }

  if (screen === 'login') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-blue-500 flex items-center justify-center p-4">
        <div className="bg-white rounded-3xl shadow-2xl p-8 w-full max-w-md">
          <div className="text-center mb-8">
            <div className="w-20 h-20 bg-gradient-to-br from-purple-500 to-pink-500 rounded-2xl mx-auto mb-4 flex items-center justify-center">
              <Target className="w-10 h-10 text-white" />
            </div>
            <h1 className="text-3xl font-bold text-gray-800">HabitFlow</h1>
            <p className="text-gray-600 mt-2">Build better habits, every day</p>
          </div>

          <div className="flex gap-2 mb-6">
            <button
              onClick={() => setAuthMode('login')}
              className={`flex-1 py-3 rounded-xl font-semibold transition-all ${
                authMode === 'login'
                  ? 'bg-gradient-to-r from-purple-500 to-pink-500 text-white'
                  : 'bg-gray-100 text-gray-600'
              }`}
            >
              Login
            </button>
            <button
              onClick={() => setAuthMode('signup')}
              className={`flex-1 py-3 rounded-xl font-semibold transition-all ${
                authMode === 'signup'
                  ? 'bg-gradient-to-r from-purple-500 to-pink-500 text-white'
                  : 'bg-gray-100 text-gray-600'
              }`}
            >
              Sign Up
            </button>
          </div>

          <div className="space-y-4">
            {authMode === 'signup' && (
              <div className="relative">
                <User className="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400" />
                <input
                  type="text"
                  value={username}
                  onChange={(e) => setUsername(e.target.value)}
                  placeholder="Username"
                  className="w-full pl-12 pr-4 py-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500"
                />
              </div>
            )}
            
            <div className="relative">
              <Mail className="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400" />
              <input
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                placeholder="Email"
                className="w-full pl-12 pr-4 py-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500"
              />
            </div>

            <div className="relative">
              <Lock className="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400" />
              <input
                type={showPassword ? 'text' : 'password'}
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                onKeyPress={(e) => e.key === 'Enter' && handleAuth()}
                placeholder="Password"
                className="w-full pl-12 pr-12 py-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500"
              />
              <button
                onClick={() => setShowPassword(!showPassword)}
                className="absolute right-3 top-1/2 transform -translate-y-1/2"
              >
                {showPassword ? <EyeOff className="w-5 h-5 text-gray-400" /> : <Eye className="w-5 h-5 text-gray-400" />}
              </button>
            </div>

            <button
              onClick={handleAuth}
              className="w-full bg-gradient-to-r from-purple-500 to-pink-500 text-white py-3 rounded-xl font-semibold hover:shadow-lg transition-all"
            >
              {authMode === 'login' ? 'Login' : 'Create Account'}
            </button>
          </div>

          <p className="text-center text-sm text-gray-600 mt-6">
            By continuing, you agree to our Terms & Privacy Policy
          </p>
        </div>
      </div>
    );
  }

  // Help Center Screen
  if (screen === 'help') {
    return (
      <div className="min-h-screen bg-gray-50">
        <div className="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6">
          <button onClick={() => setScreen('home')} className="mb-4">
            <X className="w-6 h-6" />
          </button>
          <h1 className="text-2xl font-bold">Help Center</h1>
        </div>

        <div className="max-w-2xl mx-auto p-6 space-y-4">
          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-2 flex items-center gap-2">
              <Target className="w-5 h-5 text-purple-500" />
              Getting Started
            </h3>
            <p className="text-gray-600">Create your first habit by tapping the "Add New Habit" button. Choose a name, category, color, and set your goals.</p>
          </div>

          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-2 flex items-center gap-2">
              <Check className="w-5 h-5 text-green-500" />
              Tracking Habits
            </h3>
            <p className="text-gray-600">Tap the checkbox next to any habit to mark it complete for today. Complete habits will show a green checkmark.</p>
          </div>

          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-2 flex items-center gap-2">
              <Flame className="w-5 h-5 text-orange-500" />
              Building Streaks
            </h3>
            <p className="text-gray-600">Complete habits consecutively to build streaks. The longer your streak, the more motivated you'll be!</p>
          </div>

          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-2 flex items-center gap-2">
              <Bell className="w-5 h-5 text-blue-500" />
              Reminders
            </h3>
            <p className="text-gray-600">Enable reminders when creating or editing habits to get notified at your chosen time each day.</p>
          </div>

          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-2 flex items-center gap-2">
              <Edit2 className="w-5 h-5 text-pink-500" />
              Editing Habits
            </h3>
            <p className="text-gray-600">Long press or click the edit icon on any habit to modify its details, color, or category.</p>
          </div>

          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-2">Contact Support</h3>
            <p className="text-gray-600 mb-3">Need more help? Reach out to us:</p>
            <p className="text-purple-600 font-medium">support@habitflow.app</p>
          </div>
        </div>
      </div>
    );
  }

  // Statistics Screen
  if (screen === 'statistics') {
    const totalHabits = habits.length;
    const completionRate = totalHabits > 0 ? Math.round((totalCompletedToday / totalHabits) * 100) : 0;
    const totalCompletions = habits.reduce((sum, h) => sum + h.completedDates.length, 0);

    return (
      <div className="min-h-screen bg-gray-50">
        <div className="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6">
          <button onClick={() => setScreen('home')} className="mb-4">
            <X className="w-6 h-6" />
          </button>
          <h1 className="text-2xl font-bold">Statistics</h1>
        </div>

        <div className="max-w-2xl mx-auto p-6 space-y-4">
          <div className="grid grid-cols-2 gap-4">
            <div className="bg-white rounded-2xl p-6 shadow-sm">
              <BarChart3 className="w-8 h-8 text-purple-500 mb-2" />
              <p className="text-sm text-gray-600">Completion Rate</p>
              <p className="text-3xl font-bold text-gray-800">{completionRate}%</p>
            </div>

            <div className="bg-white rounded-2xl p-6 shadow-sm">
              <Award className="w-8 h-8 text-yellow-500 mb-2" />
              <p className="text-sm text-gray-600">Total Completions</p>
              <p className="text-3xl font-bold text-gray-800">{totalCompletions}</p>
            </div>
          </div>

          <div className="bg-white rounded-2xl p-6 shadow-sm">
            <h3 className="font-semibold text-lg mb-4">Habit Breakdown</h3>
            {habits.map(habit => {
              const completions = habit.completedDates.length;
              const streak = getStreak(habit.completedDates);
              return (
                <div key={habit.id} className="mb-4 pb-4 border-b last:border-b-0">
                  <div className="flex items-center justify-between mb-2">
                    <div className="flex items-center gap-2">
                      <div className="w-3 h-3 rounded-full" style={{ backgroundColor: habit.color }}></div>
                      <span className="font-medium">{habit.name}</span>
                    </div>
                    <span className="text-sm text-gray-600">{completions} times</span>
                  </div>
                  <div className="flex items-center gap-2 text-sm text-gray-600">
                    <Flame className="w-4 h-4 text-orange-500" />
                    <span>{streak} day streak</span>
                  </div>
                </div>
              );
            })}
          </div>
        </div>
      </div>
    );
  }

  // Profile Screen
  if (screen === 'profile') {
    return (
      <div className="min-h-screen bg-gray-50">
        <div className="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6">
          <button onClick={() => setScreen('home')} className="mb-4">
            <X className="w-6 h-6" />
          </button>
          <h1 className="text-2xl font-bold">Profile</h1>
        </div>

        <div className="max-w-2xl mx-auto p-6">
          <div className="bg-white rounded-2xl p-6 shadow-sm mb-4">
            <div className="flex items-center gap-4 mb-6">
              <div className="w-20 h-20 bg-gradient-to-br from-purple-500 to-pink-500 rounded-full flex items-center justify-center text-white text-2xl font-bold">
                {currentUser?.username?.charAt(0).toUpperCase()}
              </div>
              <div>
                <h2 className="text-xl font-bold text-gray-800">{currentUser?.username}</h2>
                <p className="text-gray-600">{currentUser?.email}</p>
              </div>
            </div>

            <div className="space-y-3">
              <div className="flex items-center justify-between py-3 border-b">
                <span className="text-gray-700">Member since</span>
                <span className="font-medium">{new Date(currentUser?.createdAt).toLocaleDateString()}</span>
              </div>
              <div className="flex items-center justify-between py-3 border-b">
                <span className="text-gray-700">Total Habits</span>
                <span className="font-medium">{habits.length}</span>
              </div>
              <div className="flex items-center justify-between py-3">
                <span className="text-gray-700">Best Streak</span>
                <span className="font-medium">{bestStreak} days</span>
              </div>
            </div>
          </div>

          <button
            onClick={handleLogout}
            className="w-full bg-red-500 text-white py-4 rounded-2xl font-semibold flex items-center justify-center gap-2 hover:bg-red-600 transition-colors"
          >
            <LogOut className="w-5 h-5" />
            Logout
          </button>
        </div>
      </div>
    );
  }

  // Main Home Screen
  return (
    <div className="min-h-screen bg-gray-50 pb-20">
      {/* Header */}
      <div className="bg-gradient-to-r from-purple-500 to-pink-500 text-white p-6 sticky top-0 z-10 shadow-lg">
        <div className="flex items-center justify-between mb-4">
          <div className="flex items-center gap-3">
            <div className="w-10 h-10 bg-white bg-opacity-20 rounded-xl flex items-center justify-center">
              <Target className="w-6 h-6" />
            </div>
            <div>
              <h1 className="text-2xl font-bold">HabitFlow</h1>
              <p className="text-sm opacity-90">Hi, {currentUser?.username}!</p>
            </div>
          </div>
          <button onClick={() => setMenuOpen(true)}>
            <Menu className="w-6 h-6" />
          </button>
        </div>

        {/* Stats */}
        <div className="grid grid-cols-3 gap-3">
          <div className="bg-white bg-opacity-20 rounded-xl p-3 backdrop-blur-sm">
            <p className="text-xs opacity-90">Today</p>
            <p className="text-xl font-bold">{totalCompletedToday}/{habits.length}</p>
          </div>
          <div className="bg-white bg-opacity-20 rounded-xl p-3 backdrop-blur-sm">
            <p className="text-xs opacity-90">Streak</p>
            <p className="text-xl font-bold">{bestStreak} 🔥</p>
          </div>
          <div className="bg-white bg-opacity-20 rounded-xl p-3 backdrop-blur-sm">
            <p className="text-xs opacity-90">Total</p>
            <p className="text-xl font-bold">{habits.length}</p>
          </div>
        </div>
      </div>

      {/* Habits List */}
      <div className="p-6 space-y-4">
        <h2 className="text-lg font-semibold text-gray-800 mb-4">Today's Habits</h2>
        
        {habits.length === 0 ? (
          <div className="bg-white rounded-2xl p-12 text-center shadow-sm">
            <Target className="w-16 h-16 text-gray-300 mx-auto mb-4" />
            <h3 className="text-lg font-semibold text-gray-800 mb-2">No habits yet</h3>
            <p className="text-gray-600 mb-4">Start building better habits today!</p>
          </div>
        ) : (
          habits.map(habit => {
            const isCompletedToday = habit.completedDates.includes(today);
            const streak = getStreak(habit.completedDates);

            return (
              <div
                key={habit.id}
                className="bg-white rounded-2xl p-4 shadow-sm border-l-4"
                style={{ borderLeftColor: habit.color }}
              >
                <div className="flex items-center gap-3">
                  <button
                    onClick={() => toggleHabit(habit.id)}
                    className={`w-14 h-14 rounded-xl flex-shrink-0 flex items-center justify-center transition-all ${
                      isCompletedToday
                        ? 'bg-gradient-to-br from-green-400 to-green-600 shadow-lg scale-105'
                        : 'bg-gray-100 hover:bg-gray-200'
                    }`}
                  >
                    {isCompletedToday && <Check className="w-7 h-7 text-white" />}
                  </button>

                  <div className="flex-1 min-w-0">
                    <div className="flex items-center gap-2 mb-1">
                      <span className="text-xl">{getCategoryIcon(habit.category)}</span>
                      <h3 className="text-lg font-semibold text-gray-800 truncate">{habit.name}</h3>
                    </div>
                    <div className="flex items-center gap-3 text-sm text-gray-600">
                      <div className="flex items-center gap-1">
                        <Flame className="w-4 h-4 text-orange-500" />
                        <span>{streak} days</span>
                      </div>
                      <span>•</span>
                      <span className="capitalize">{habit.goal}</span>
                      {habit.reminder && (
                        <>
                          <span>•</span>
                          <Bell className="w-4 h-4" />
                        </>
                      )}
                    </div>
                  </div>

                  <div className="flex gap-2">
                    <button
                      onClick={() => startEditHabit(habit)}
                      className="p-2 hover:bg-blue-50 rounded-lg transition-colors"
                    >
                      <Edit2 className="w-5 h-5 text-blue-500" />
                    </button>
                    <button
                      onClick={() => deleteHabit(habit.id)}
                      className="p-2 hover:bg-red-50 rounded-lg transition-colors"
                    >
                      <Trash2 className="w-5 h-5 text-red-500" />
                    </button>
                  </div>
                </div>

                {habit.notes && (
                  <p className="mt-3 text-sm text-gray-600 pl-17">{habit.notes}</p>
                )}
              </div>
            );
          })
        )}
      </div>

      {/* Add Habit Button */}
      {!isAddingHabit && (
        <button
          onClick={() => setIsAddingHabit(true)}
          className="fixed bottom-24 right-6 w-16 h-16 bg-gradient-to-r from-purple-500 to-pink-500 text-white rounded-full shadow-2xl flex items-center justify-center hover:scale-110 transition-transform z-20"
        >
          <Plus className="w-8 h-8" />
        </button>
      )}

      {/* Add/Edit Habit Modal */}
      {isAddingHabit && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-end justify-center z-50 p-4">
          <div className="bg-white rounded-t-3xl w-full max-w-2xl max-h-[85vh] overflow-y-auto">
            <div className="sticky top-0 bg-white p-6 border-b flex items-center justify-between">
              <h2 className="text-xl font-bold text-gray-800">
                {editingHabit ? 'Edit Habit' : 'Create New Habit'}
              </h2>
              <button onClick={resetHabitForm}>
                <X className="w-6 h-6 text-gray-600" />
              </button>
            </div>

            <div className="p-6 space-y-6">
              {/* Habit Name */}
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-2">Habit Name</label>
                <input
                  type="text"
                  value={newHabit.name}
                  onChange={(e) => setNewHabit({ ...newHabit, name: e.target.value })}
                  placeholder="e.g., Morning meditation"
                  className="w-full p-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500"
                />
              </div>

              {/* Category */}
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-2">Category</label>
                <div className="grid grid-cols-4 gap-2">
                  {categories.map(cat => (
                    <button
                      key={cat}
                      onClick={() => setNewHabit({ ...newHabit, category: cat })}
                      className={`p-3 rounded-xl text-center transition-all ${
                        newHabit.category === cat
                          ? 'bg-purple-500 text-white shadow-lg scale-105'
                          : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                      }`}
                    >
                      <div className="text-2xl mb-1">{getCategoryIcon(cat)}</div>
                      <div className="text-xs capitalize">{cat}</div>
                    </button>
                  ))}
                </div>
              </div>

              {/* Color */}
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-2">Color</label>
                <div className="flex gap-3 flex-wrap">
                  {colors.map(color => (
                    <button
                      key={color}
                      onClick={() => setNewHabit({ ...newHabit, color })}
                      className={`w-12 h-12 rounded-xl transition-all ${
                        newHabit.color === color ? 'ring-4 ring-offset-2 ring-gray-400 scale-110' : ''
                      }`}
                      style={{ backgroundColor: color }}
                    />
                  ))}
                </div>
              </div>

              {/* Goal Type */}
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-2">Goal Frequency</label>
                <div className="grid grid-cols-3 gap-2">
                  {goalTypes.map(goal => (
                    <button
                      key={goal}
                      onClick={() => setNewHabit({ ...newHabit, goal })}
                      className={`p-3 rounded-xl text-center transition-all capitalize ${
                        newHabit.goal === goal
                          ? 'bg-purple-500 text-white'
                          : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
                      }`}
                    >
                      {goal}
                    </button>
                  ))}
                </div>
              </div>

              {/* Reminder */}
              <div>
                <div className="flex items-center justify-between mb-2">
                  <label className="block text-sm font-medium text-gray-700">Daily Reminder</label>
                  <button
                    onClick={() => setNewHabit({ ...newHabit, reminder: !newHabit.reminder })}
                    className={`w-12 h-6 rounded-full transition-all ${
                      newHabit.reminder ? 'bg-purple-500' : 'bg-gray-300'
                    }`}
                  >
                    <div className={`w-5 h-5 bg-white rounded-full shadow transition-transform ${
                      newHabit.reminder ? 'translate-x-6' : 'translate-x-0.5'
                    }`}></div>
                  </button>
                </div>
                {newHabit.reminder && (
                  <input
                    type="time"
                    value={newHabit.reminderTime}
                    onChange={(e) => setNewHabit({ ...newHabit, reminderTime: e.target.value })}
                    className="w-full p-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500"
                  />
                )}
              </div>

              {/* Notes */}
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-2">Notes (Optional)</label>
                <textarea
                  value={newHabit.notes}
                  onChange={(e) => setNewHabit({ ...newHabit, notes: e.target.value })}
                  placeholder="Add any notes or motivation..."
                  className="w-full p-3 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500 min-h-24"
                />
              </div>

              {/* Action Buttons */}
              <div className="flex gap-3">
                <button
                  onClick={addOrUpdateHabit}
                  className="flex-1 bg-gradient-to-r from-purple-500 to-pink-500 text-white py-4 rounded-xl font-semibold hover:shadow-lg transition-all"
                >
                  {editingHabit ? 'Update Habit' : 'Create Habit'}
                </button>
                <button
                  onClick={resetHabitForm}
                  className="px-8 bg-gray-100 text-gray-700 py-4 rounded-xl font-semibold hover:bg-gray-200 transition-colors"
                >
                  Cancel
                </button>
              </div>
            </div>
          </div>
        </div>
      )}

      {/* Side Menu */}
      {menuOpen && (
        <div className="fixed inset-0 z-50">
          <div className="absolute inset-0 bg-black bg-opacity-50" onClick={() => setMenuOpen(false)}></div>
          <div className="absolute right-0 top-0 bottom-0 w-80 bg-white shadow-2xl">
            <div className="p-6 bg-gradient-to-r from-purple-500 to-pink-500 text-white">
              <div className="flex items-center justify-between mb-4">
                <h2 className="text-xl font-bold">Menu</h2>
                <button onClick={() => setMenuOpen(false)}>
                  <X className="w-6 h-6" />
                </button>
              </div>
              <div className="flex items-center gap-3">
                <div className="w-12 h-12 bg-white bg-opacity-20 rounded-full flex items-center justify-center text-lg font-bold">
                  {currentUser?.username?.charAt(0).toUpperCase()}
                </div>
                <div>
                  <p className="font-semibold">{currentUser?.username}</p>
                  <p className="text-sm opacity-90">{currentUser?.email}</p>
                </div>
              </div>
            </div>

            <div className="p-4 space-y-2">
              <button
                onClick={() => { setScreen('profile'); setMenuOpen(false); }}
                className="w-full flex items-center gap-3 p-4 rounded-xl hover:bg-gray-100 transition-colors"
              >
                <User className="w-5 h-5 text-gray-600" />
                <span className="font-medium text-gray-800">Profile</span>
                <ChevronRight className="w-5 h-5 text-gray-400 ml-auto" />
              </button>

              <button
                onClick={() => { setScreen('statistics'); setMenuOpen(false); }}
                className="w-full flex items-center gap-3 p-4 rounded-xl hover:bg-gray-100 transition-colors"
              >
                <BarChart3 className="w-5 h-5 text-gray-600" />
                <span className="font-medium text-gray-800">Statistics</span>
                <ChevronRight className="w-5 h-5 text-gray-400 ml-auto" />
              </button>

              <button
                onClick={() => { setScreen('help'); setMenuOpen(false); }}
                className="w-full flex items-center gap-3 p-4 rounded-xl hover:bg-gray-100 transition-colors"
              >
                <HelpCircle className="w-5 h-5 text-gray-600" />
                <span className="font-medium text-gray-800">Help Center</span>
                <ChevronRight className="w-5 h-5 text-gray-400 ml-auto" />
              </button>

              <button
                className="w-full flex items-center gap-3 p-4 rounded-xl hover:bg-gray-100 transition-colors"
              >
                <Bell className="w-5 h-5 text-gray-600" />
                <span className="font-medium text-gray-800">Notifications</span>
                <ChevronRight className="w-5 h-5 text-gray-400 ml-auto" />
              </button>

              <button
                className="w-full flex items-center gap-3 p-4 rounded-xl hover:bg-gray-100 transition-colors"
              >
                <Settings className="w-5 h-5 text-gray-600" />
                <span className="font-medium text-gray-800">Settings</span>
                <ChevronRight className="w-5 h-5 text-gray-400 ml-auto" />
              </button>
            </div>

            <div className="absolute bottom-0 left-0 right-0 p-4 border-t">
              <button
                onClick={handleLogout}
                className="w-full flex items-center justify-center gap-2 p-4 rounded-xl bg-red-50 text-red-600 font-medium hover:bg-red-100 transition-colors"
              >
                <LogOut className="w-5 h-5" />
                Logout
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Bottom Navigation */}
      <div className="fixed bottom-0 left-0 right-0 bg-white border-t shadow-lg">
        <div className="flex items-center justify-around p-4 max-w-2xl mx-auto">
          <button
            onClick={() => setScreen('home')}
            className={`flex flex-col items-center gap-1 ${
              screen === 'home' ? 'text-purple-600' : 'text-gray-400'
            }`}
          >
            <Target className="w-6 h-6" />
            <span className="text-xs font-medium">Home</span>
          </button>

          <button
            onClick={() => setScreen('statistics')}
            className={`flex flex-col items-center gap-1 ${
              screen === 'statistics' ? 'text-purple-600' : 'text-gray-400'
            }`}
          >
            <BarChart3 className="w-6 h-6" />
            <span className="text-xs font-medium">Stats</span>
          </button>

          <button
            onClick={() => setIsAddingHabit(true)}
            className="flex flex-col items-center gap-1"
          >
            <div className="w-12 h-12 bg-gradient-to-r from-purple-500 to-pink-500 rounded-full flex items-center justify-center -mt-6 shadow-lg">
              <Plus className="w-6 h-6 text-white" />
            </div>
          </button>

          <button
            onClick={() => setScreen('help')}
            className={`flex flex-col items-center gap-1 ${
              screen === 'help' ? 'text-purple-600' : 'text-gray-400'
            }`}
          >
            <HelpCircle className="w-6 h-6" />
            <span className="text-xs font-medium">Help</span>
          </button>

          <button
            onClick={() => setScreen('profile')}
            className={`flex flex-col items-center gap-1 ${
              screen === 'profile' ? 'text-purple-600' : 'text-gray-400'
            }`}
          >
            <User className="w-6 h-6" />
            <span className="text-xs font-medium">Profile</span>
          </button>
        </div>
      </div>
    </div>
  );
};

export default HabitFlowApp;
