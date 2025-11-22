import React, { useState, useEffect } from 'react';
import { Heart, Utensils, Sparkles, MessageCircle, Star, Clock, Play, Pause, RotateCcw, Award, LogOut, User, Gift, Gamepad2, ShoppingBag, Trophy, Moon, Sun } from 'lucide-react';

export default function PixelPet() {
  const [page, setPage] = useState('auth');
  const [authMode, setAuthMode] = useState('login');
  const [currentUser, setCurrentUser] = useState(null);
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [authError, setAuthError] = useState('');
  const [selectedPet, setSelectedPet] = useState(null);
  const [petName, setPetName] = useState('');
  const [petStats, setPetStats] = useState({ hunger: 80, happiness: 80, health: 100 });
  const [petResponse, setPetResponse] = useState('');
  const [coins, setCoins] = useState(100);
  const [pomodoroTime, setPomodoroTime] = useState(25 * 60);
  const [pomodoroRunning, setPomodoroRunning] = useState(false);
  const [pomodoroCompleted, setPomodoroCompleted] = useState(0);
  const [foodRewards, setFoodRewards] = useState(0);
  const [ownedItems, setOwnedItems] = useState([]);
  const [equipped, setEquipped] = useState({ hat: null, acc: null });
  const [modal, setModal] = useState(null);
  const [gameScore, setGameScore] = useState(0);
  const [gameActive, setGameActive] = useState(false);
  const [targetPos, setTargetPos] = useState({ x: 50, y: 50 });
  const [signInStreak, setSignInStreak] = useState(0);
  const [lastSignIn, setLastSignIn] = useState('');
  const [chatMsg, setChatMsg] = useState('');
  const [chatHistory, setChatHistory] = useState([]);

  const isNight = new Date().getHours() >= 19 || new Date().getHours() < 6;
  const pets = [
    { id: 'cat', name: '小猫咪', emoji: '🐱' },
    { id: 'dog', name: '小狗狗', emoji: '🐶' },
    { id: 'rabbit', name: '小兔兔', emoji: '🐰' },
    { id: 'bear', name: '小熊熊', emoji: '🐻' },
  ];
  const shop = [
    { id: 'h1', name: '皇冠', emoji: '👑', type: 'hat', price: 80 },
    { id: 'h2', name: '派对帽', emoji: '🎩', type: 'hat', price: 50 },
    { id: 'h3', name: '花环', emoji: '💐', type: 'hat', price: 60 },
    { id: 'a1', name: '墨镜', emoji: '🕶️', type: 'acc', price: 60 },
    { id: 'a2', name: '蝴蝶结', emoji: '🎀', type: 'acc', price: 40 },
    { id: 'a3', name: '围巾', emoji: '🧣', type: 'acc', price: 55 },
  ];

  useEffect(() => {
    (async () => {
      try {
        const s = await window.storage.get('session');
        if (s?.value) {
          const d = JSON.parse(s.value);
          await loadUser(d.user);
        }
      } catch(e) {}
    })();
  }, []);

  const loadUser = async (u) => {
    try {
      const d = await window.storage.get('data_' + u);
      if (d?.value) {
        const p = JSON.parse(d.value);
        setCurrentUser(u); setSelectedPet(p.pet); setPetName(p.name || '');
        setPetStats(p.stats || { hunger: 80, happiness: 80, health: 100 });
        setCoins(p.coins || 100); setPomodoroCompleted(p.pomo || 0);
        setFoodRewards(p.food || 0); setOwnedItems(p.owned || []);
        setEquipped(p.equipped || { hat: null, acc: null });
        setSignInStreak(p.streak || 0); setLastSignIn(p.lastSign || '');
        setChatHistory(p.chat || []);
        setPage(p.pet ? 'game' : 'select');
        const today = new Date().toDateString();
        if (p.lastSign !== today) setModal('signin');
      } else {
        setCurrentUser(u); setPage('select');
      }
    } catch(e) { setCurrentUser(u); setPage('select'); }
  };

  const save = async () => {
    if (!currentUser) return;
    const d = { pet: selectedPet, name: petName, stats: petStats, coins, pomo: pomodoroCompleted, food: foodRewards, owned: ownedItems, equipped, streak: signInStreak, lastSign: lastSignIn, chat: chatHistory };
    try { await window.storage.set('data_' + currentUser, JSON.stringify(d)); } catch(e) {}
  };

  useEffect(() => { if (currentUser) { const t = setTimeout(save, 500); return () => clearTimeout(t); } }, [petStats, coins, pomodoroCompleted, foodRewards, ownedItems, equipped, signInStreak, lastSignIn, chatHistory, selectedPet, petName]);

  const register = async () => {
    if (username.length < 2) { setAuthError('用户名至少2字符'); return; }
    if (password.length < 4) { setAuthError('密码至少4位'); return; }
    try {
      const e = await window.storage.get('user_' + username).catch(() => null);
      if (e?.value) { setAuthError('用户已存在'); return; }
      await window.storage.set('user_' + username, JSON.stringify({ pw: password }));
      await window.storage.set('session', JSON.stringify({ user: username }));
      setCurrentUser(username); setPage('select'); setUsername(''); setPassword('');
    } catch(e) { setAuthError('注册失败'); }
  };

  const login = async () => {
    if (!username || !password) { setAuthError('请填写完整'); return; }
    try {
      const u = await window.storage.get('user_' + username).catch(() => null);
      if (!u?.value) { setAuthError('用户不存在'); return; }
      if (JSON.parse(u.value).pw !== password) { setAuthError('密码错误'); return; }
      await window.storage.set('session', JSON.stringify({ user: username }));
      await loadUser(username); setUsername(''); setPassword('');
    } catch(e) { setAuthError('登录失败'); }
  };

  const logout = async () => {
    await save();
    try { await window.storage.delete('session'); } catch(e) {}
    setCurrentUser(null); setPage('auth'); setSelectedPet(null); setPetName('');
    setPetStats({ hunger: 80, happiness: 80, health: 100 }); setCoins(100);
    setPomodoroCompleted(0); setFoodRewards(0); setOwnedItems([]);
    setEquipped({ hat: null, acc: null }); setChatHistory([]);
  };

  useEffect(() => {
    if (page !== 'game') return;
    const i = setInterval(() => setPetStats(p => ({ hunger: Math.max(0, p.hunger - 0.8), happiness: Math.max(0, p.happiness - 0.4), health: Math.max(0, p.health - 0.2) })), 5000);
    return () => clearInterval(i);
  }, [page]);

  useEffect(() => {
    if (!pomodoroRunning) return;
    const t = setInterval(() => {
      setPomodoroTime(p => {
        if (p <= 1) {
          setPomodoroRunning(false);
          setPomodoroCompleted(c => c + 1);
          setFoodRewards(f => f + 1);
          setCoins(c => c + 30);
          showMsg('番茄钟完成! +30金币 🎉');
          return 25 * 60;
        }
        return p - 1;
      });
    }, 1000);
    return () => clearInterval(t);
  }, [pomodoroRunning]);

  useEffect(() => {
    if (!gameActive) return;
    const t = setTimeout(() => {
      setGameActive(false);
      setCoins(c => c + gameScore * 2);
      showMsg(`游戏结束! +${gameScore * 2}金币 🎮`);
    }, 10000);
    return () => clearTimeout(t);
  }, [gameActive]);

  const showMsg = (m) => { setPetResponse(m); setTimeout(() => setPetResponse(''), 3000); };
  const feed = () => { setPetStats(p => ({ ...p, hunger: Math.min(100, p.hunger + 20), happiness: Math.min(100, p.happiness + 5) })); showMsg('好吃! 😋'); };
  const feedReward = () => { if (foodRewards <= 0) return; setFoodRewards(f => f - 1); setPetStats(p => ({ ...p, hunger: Math.min(100, p.hunger + 35), happiness: Math.min(100, p.happiness + 15) })); showMsg('奖励美食太棒了! 🌟'); };
  const playPet = () => { setPetStats(p => ({ ...p, happiness: Math.min(100, p.happiness + 20), health: Math.min(100, p.health + 5) })); showMsg('好开心! 🎉'); };
  const rest = () => { setPetStats(p => ({ ...p, health: Math.min(100, p.health + 20), happiness: Math.min(100, p.happiness + 5) })); showMsg('休息好了! 😴'); };

  const signIn = () => {
    const today = new Date().toDateString();
    const yest = new Date(Date.now() - 86400000).toDateString();
    const streak = lastSignIn === yest ? signInStreak + 1 : 1;
    const reward = Math.min(10 + streak * 5, 50);
    setSignInStreak(streak); setLastSignIn(today); setCoins(c => c + reward);
    setModal(null); showMsg(`签到成功! 连续${streak}天 +${reward}金币 🎁`);
  };

  const buyItem = (item) => {
    if (coins < item.price || ownedItems.includes(item.id)) return;
    setCoins(c => c - item.price);
    setOwnedItems(o => [...o, item.id]);
    showMsg(`购买${item.name}成功! 🛍️`);
  };

  const equipItem = (item) => {
    if (!ownedItems.includes(item.id)) return;
    setEquipped(e => ({ ...e, [item.type]: e[item.type] === item.id ? null : item.id }));
  };

  const hitTarget = () => {
    if (!gameActive) return;
    setGameScore(s => s + 1);
    setTargetPos({ x: Math.random() * 70 + 15, y: Math.random() * 60 + 20 });
  };

  const sendChat = () => {
    if (!chatMsg.trim()) return;
    setChatHistory(h => [...h, { t: 'u', m: chatMsg }]);
    const m = chatMsg.toLowerCase();
    setTimeout(() => {
      let r = '嗯嗯~ 😊';
      if (m.includes('你好')) r = '你好呀! 😊';
      else if (m.includes('爱') || m.includes('喜欢')) r = '我也爱你! ❤️';
      else if (m.includes('金币')) r = `你有${coins}金币哦! 💰`;
      else if (m.includes('饿')) r = petStats.hunger < 30 ? '我好饿! 🥺' : '我不饿~ 😋';
      setChatHistory(h => [...h, { t: 'p', m: r }]);
    }, 500);
    setChatMsg('');
  };

  const mood = (petStats.hunger + petStats.happiness + petStats.health) / 3 > 60 ? '😊' : '😢';
  const getEmoji = (type) => shop.find(i => i.id === equipped[type])?.emoji;
  const bg = isNight ? 'from-indigo-900 via-purple-900 to-blue-900' : 'from-pink-50 via-blue-50 to-purple-50';
  const fmt = (s) => `${Math.floor(s/60).toString().padStart(2,'0')}:${(s%60).toString().padStart(2,'0')}`;

  // 登录页
  if (page === 'auth') return (
    <div className={`min-h-screen bg-gradient-to-br ${bg} flex items-center justify-center p-4`}>
      <div className="bg-white/90 rounded-3xl shadow-2xl p-8 max-w-md w-full">
        <div className="text-center mb-2">{isNight ? <Moon className="inline text-yellow-300" /> : <Sun className="inline text-yellow-500" />}</div>
        <h1 className="text-3xl font-bold text-center mb-2 bg-gradient-to-r from-pink-400 to-purple-400 bg-clip-text text-transparent">AI电子宠物小站</h1>
        <p className="text-center text-gray-500 mb-6">登录后数据自动保存 ✨</p>
        <div className="flex mb-6 bg-gray-100 rounded-xl p-1">
          <button onClick={() => setAuthMode('login')} className={`flex-1 py-2 rounded-lg ${authMode === 'login' ? 'bg-white shadow' : ''}`}>登录</button>
          <button onClick={() => setAuthMode('register')} className={`flex-1 py-2 rounded-lg ${authMode === 'register' ? 'bg-white shadow' : ''}`}>注册</button>
        </div>
        <input value={username} onChange={e => setUsername(e.target.value)} placeholder="用户名 (2-20字符)" className="w-full px-4 py-3 rounded-xl border-2 border-gray-200 mb-3" maxLength={20} />
        <input type="password" value={password} onChange={e => setPassword(e.target.value)} placeholder="密码 (4-20位)" className="w-full px-4 py-3 rounded-xl border-2 border-gray-200 mb-3" maxLength={20} onKeyDown={e => e.key === 'Enter' && (authMode === 'login' ? login() : register())} />
        {authError && <p className="text-red-500 text-sm text-center mb-3">{authError}</p>}
        <button onClick={authMode === 'login' ? login : register} className="w-full py-3 rounded-xl bg-gradient-to-r from-pink-400 to-purple-400 text-white font-bold">{authMode === 'login' ? '登录' : '注册'}</button>
      </div>
    </div>
  );

  // 选宠物页
  if (page === 'select') return (
    <div className={`min-h-screen bg-gradient-to-br ${bg} flex items-center justify-center p-4`}>
      <div className="bg-white/90 rounded-3xl shadow-2xl p-8 max-w-lg w-full">
        <div className="flex justify-between mb-6">
          <h1 className="text-2xl font-bold bg-gradient-to-r from-pink-400 to-purple-400 bg-clip-text text-transparent">选择宠物</h1>
          <button onClick={logout} className="text-gray-500 flex items-center gap-1"><LogOut size={18} />登出</button>
        </div>
        {!selectedPet ? (
          <div className="grid grid-cols-2 gap-4">
            {pets.map(p => <button key={p.id} onClick={() => setSelectedPet(p)} className="p-6 rounded-2xl border-2 hover:border-purple-300 hover:shadow-lg bg-white"><div className="text-5xl mb-2">{p.emoji}</div><div className="font-semibold">{p.name}</div></button>)}
          </div>
        ) : (
          <div className="text-center">
            <div className="text-7xl mb-4">{selectedPet.emoji}</div>
            <input value={petName} onChange={e => setPetName(e.target.value)} placeholder="起个名字..." className="px-4 py-3 rounded-xl border-2 w-full max-w-xs mb-4" />
            <div className="flex gap-3 justify-center">
              <button onClick={() => setSelectedPet(null)} className="px-6 py-2 rounded-xl bg-gray-200">返回</button>
              <button onClick={() => { if (petName.trim()) { setPage('game'); setChatHistory([{ t: 'p', m: `你好!我是${petName}~ 🎉` }]); } }} className="px-6 py-2 rounded-xl bg-gradient-to-r from-pink-400 to-purple-400 text-white">开始!</button>
            </div>
          </div>
        )}
      </div>
    </div>
  );

  // 游戏主页
  return (
    <div className={`min-h-screen bg-gradient-to-br ${bg} p-3`}>
      {/* 签到弹窗 */}
      {modal === 'signin' && <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50"><div className="bg-white rounded-2xl p-6 text-center"><Gift className="mx-auto text-pink-500 mb-2" size={40} /><h3 className="text-xl font-bold mb-2">每日签到</h3><p className="text-gray-600 mb-4">连续第{signInStreak + 1}天<br/>可得{Math.min(15 + signInStreak * 5, 50)}金币</p><button onClick={signIn} className="px-6 py-2 rounded-xl bg-gradient-to-r from-pink-400 to-purple-400 text-white">签到 🎁</button></div></div>}
      
      {/* 商店弹窗 */}
      {modal === 'shop' && <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"><div className="bg-white rounded-2xl p-5 max-w-md w-full max-h-96 overflow-auto"><div className="flex justify-between mb-4"><h3 className="text-xl font-bold">🛍️ 商店</h3><button onClick={() => setModal(null)} className="text-gray-500">✕</button></div><div className="grid grid-cols-3 gap-3">{shop.map(i => <div key={i.id} className={`p-3 rounded-xl text-center border-2 ${ownedItems.includes(i.id) ? 'border-green-300 bg-green-50' : 'border-gray-200'}`}><div className="text-3xl">{i.emoji}</div><div className="text-sm">{i.name}</div>{!ownedItems.includes(i.id) ? <button onClick={() => buyItem(i)} disabled={coins < i.price} className="mt-1 px-2 py-1 text-xs rounded bg-yellow-400 text-white disabled:opacity-50">{i.price}币</button> : <button onClick={() => equipItem(i)} className={`mt-1 px-2 py-1 text-xs rounded ${equipped[i.type] === i.id ? 'bg-pink-400 text-white' : 'bg-gray-200'}`}>{equipped[i.type] === i.id ? '卸下' : '装备'}</button>}</div>)}</div></div></div>}
      
      {/* 小游戏弹窗 */}
      {modal === 'game' && <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"><div className="bg-white rounded-2xl p-5 w-80"><div className="flex justify-between mb-3"><h3 className="font-bold">🎮 点点乐 (10秒)</h3><button onClick={() => { setModal(null); setGameActive(false); }} className="text-gray-500">✕</button></div>{!gameActive ? <div className="text-center"><p className="mb-3">点击目标得分!<br/>每分=2金币</p><button onClick={() => { setGameScore(0); setGameActive(true); setTargetPos({ x: 50, y: 50 }); }} className="px-6 py-2 rounded-xl bg-green-400 text-white">开始!</button></div> : <div className="relative h-48 bg-gray-100 rounded-xl"><div className="absolute text-2xl cursor-pointer select-none" style={{ left: targetPos.x + '%', top: targetPos.y + '%', transform: 'translate(-50%,-50%)' }} onClick={hitTarget}>🎯</div><div className="absolute top-2 right-2 bg-white px-2 rounded">分数: {gameScore}</div></div>}</div></div>}
      
      {/* 聊天弹窗 */}
      {modal === 'chat' && <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4"><div className="bg-white rounded-2xl p-4 w-full max-w-md"><div className="flex justify-between mb-3"><h3 className="font-bold">💬 聊天</h3><button onClick={() => setModal(null)} className="text-gray-500">✕</button></div><div className="h-48 overflow-auto mb-3 space-y-2">{chatHistory.map((c, i) => <div key={i} className={`flex ${c.t === 'u' ? 'justify-end' : ''}`}><div className={`px-3 py-2 rounded-xl max-w-xs ${c.t === 'u' ? 'bg-purple-400 text-white' : 'bg-gray-100'}`}>{c.m}</div></div>)}</div><div className="flex gap-2"><input value={chatMsg} onChange={e => setChatMsg(e.target.value)} onKeyDown={e => e.key === 'Enter' && sendChat()} className="flex-1 px-3 py-2 rounded-xl border-2" placeholder="说点什么..." /><button onClick={sendChat} className="px-4 py-2 rounded-xl bg-purple-400 text-white">发送</button></div></div></div>}
      
      {/* 番茄钟弹窗 */}
      {modal === 'pomo' && <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50"><div className="bg-white rounded-2xl p-6 text-center"><h3 className="text-xl font-bold mb-3">🍅 番茄钟</h3><div className="text-5xl font-bold text-green-600 mb-3">{fmt(pomodoroTime)}</div><div className="flex gap-3 justify-center mb-3">{!pomodoroRunning ? <button onClick={() => setPomodoroRunning(true)} className="px-5 py-2 rounded-xl bg-green-400 text-white flex items-center gap-1"><Play size={18} />开始</button> : <button onClick={() => setPomodoroRunning(false)} className="px-5 py-2 rounded-xl bg-orange-400 text-white flex items-center gap-1"><Pause size={18} />暂停</button>}<button onClick={() => { setPomodoroRunning(false); setPomodoroTime(25*60); }} className="px-5 py-2 rounded-xl bg-gray-200 flex items-center gap-1"><RotateCcw size={18} />重置</button></div><p className="text-sm text-gray-500">完成+30金币+1食物</p><button onClick={() => setModal(null)} className="mt-3 text-gray-500">关闭</button></div></div>}

      <div className="max-w-lg mx-auto">
        {/* 顶栏 */}
        <div className="flex justify-between items-center mb-3">
          <h1 className={`text-xl font-bold ${isNight ? 'text-white' : 'text-purple-600'}`}>{petName}的小窝 {isNight ? '🌙' : '☀️'}</h1>
          <div className="flex items-center gap-2">
            <span className="bg-white/90 px-3 py-1 rounded-full text-sm font-bold text-yellow-600">💰{coins}</span>
            <button onClick={logout} className="p-2 bg-white/90 rounded-full"><LogOut size={16} /></button>
          </div>
        </div>

        {/* 状态条 */}
        <div className="bg-white/90 rounded-2xl p-3 mb-3 shadow">
          <div className="grid grid-cols-3 gap-3 text-sm">
            <div><div className="flex items-center gap-1 mb-1"><Utensils size={14} className="text-orange-400" />饱食</div><div className="h-2 bg-gray-200 rounded-full"><div className="h-full bg-orange-400 rounded-full" style={{ width: petStats.hunger + '%' }} /></div></div>
            <div><div className="flex items-center gap-1 mb-1"><Sparkles size={14} className="text-pink-400" />快乐</div><div className="h-2 bg-gray-200 rounded-full"><div className="h-full bg-pink-400 rounded-full" style={{ width: petStats.happiness + '%' }} /></div></div>
            <div><div className="flex items-center gap-1 mb-1"><Heart size={14} className="text-red-400" />健康</div><div className="h-2 bg-gray-200 rounded-full"><div className="h-full bg-red-400 rounded-full" style={{ width: petStats.health + '%' }} /></div></div>
          </div>
        </div>

        {/* 宠物展示 */}
        <div className="bg-white/90 rounded-2xl p-6 mb-3 shadow relative text-center" style={{ minHeight: 200 }}>
          {petResponse && <div className="absolute top-3 left-1/2 -translate-x-1/2 bg-yellow-100 px-4 py-2 rounded-full text-sm shadow">{petResponse}</div>}
          <div className="relative inline-block">
            {getEmoji('hat') && <span className="absolute -top-6 left-1/2 -translate-x-1/2 text-3xl">{getEmoji('hat')}</span>}
            <span className="text-8xl">{selectedPet?.emoji}</span>
            {getEmoji('acc') && <span className="absolute -bottom-2 left-1/2 -translate-x-1/2 text-2xl">{getEmoji('acc')}</span>}
          </div>
          <div className="mt-3 text-lg font-bold text-gray-700">{petName} {mood}</div>
          <div className="text-sm text-gray-500">🍅完成{pomodoroCompleted} | 🍖食物{foodRewards}</div>
        </div>

        {/* 操作按钮 */}
        <div className="grid grid-cols-4 gap-2 mb-3">
          <button onClick={feed} className="bg-white/90 p-3 rounded-xl shadow flex flex-col items-center"><Utensils size={20} className="text-orange-400" /><span className="text-xs mt-1">喂食</span></button>
          <button onClick={feedReward} disabled={foodRewards < 1} className="bg-white/90 p-3 rounded-xl shadow flex flex-col items-center disabled:opacity-50"><span className="text-lg">🍖</span><span className="text-xs">{foodRewards}</span></button>
          <button onClick={playPet} className="bg-white/90 p-3 rounded-xl shadow flex flex-col items-center"><Sparkles size={20} className="text-pink-400" /><span className="text-xs mt-1">玩耍</span></button>
          <button onClick={rest} className="bg-white/90 p-3 rounded-xl shadow flex flex-col items-center"><Heart size={20} className="text-blue-400" /><span className="text-xs mt-1">休息</span></button>
        </div>

        {/* 功能按钮 */}
        <div className="grid grid-cols-4 gap-2">
          <button onClick={() => setModal('pomo')} className="bg-green-100 p-3 rounded-xl shadow flex flex-col items-center"><Clock size={20} className="text-green-500" /><span className="text-xs mt-1">番茄钟</span></button>
          <button onClick={() => setModal('shop')} className="bg-yellow-100 p-3 rounded-xl shadow flex flex-col items-center"><ShoppingBag size={20} className="text-yellow-600" /><span className="text-xs mt-1">商店</span></button>
          <button onClick={() => setModal('game')} className="bg-purple-100 p-3 rounded-xl shadow flex flex-col items-center"><Gamepad2 size={20} className="text-purple-500" /><span className="text-xs mt-1">小游戏</span></button>
          <button onClick={() => setModal('chat')} className="bg-pink-100 p-3 rounded-xl shadow flex flex-col items-center"><MessageCircle size={20} className="text-pink-500" /><span className="text-xs mt-1">聊天</span></button>
        </div>
      </div>
    </div>
  );
}
