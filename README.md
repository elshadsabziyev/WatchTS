import React, { useState, useRef, useEffect } from 'react';
import { Search, Film, Play, AlertCircle, Loader2, Info, Sparkles, Wand2, Settings, X, Moon, Sun, MessageCircle, Send, Image as ImageIcon, ExternalLink } from 'lucide-react';

const apiKey = import.meta.env.VITE_GEMINI_API_KEY; // API key will be injected by the environment at runtime

// --- Translations Dictionary ---
const locales = {
  en: {
    langName: "English",
    titleMode: "Exact Title",
    vibeMode: "✨ AI Vibe Search",
    placeholderTitle: "e.g. Inception, Breaking Bad...",
    placeholderVibe: "e.g. A group of friends get lost in a spooky forest...",
    searchBtn: "Search",
    suggestBtn: "✨ Suggest",
    noResults: "No movies or series found.",
    tryVibe: "Try describing the plot with more detail.",
    topMatches: "Top Matches",
    otherMedia: "Other Media (Games, Episodes, etc.)",
    info: "Info",
    aiPitch: "AI Pitch",
    playMovie: "Play",
    year: "Year",
    genre: "Genre",
    director: "Director / Developer",
    cast: "Main Cast",
    unknown: "Unknown",
    gathering: "Gathering details...",
    settings: "Settings",
    language: "Language",
    theme: "Theme",
    light: "Light",
    dark: "Dark",
    close: "Close",
    aiPitchTitle: "✨ AI Pitch & Trivia",
    subtitle: "Search by exact title or describe the vibe you're looking for, and we'll find the direct stream link.",
    errorAI: "Failed to generate AI response. Please try again.",
    errorInfo: "Failed to load movie info.",
    errorSearch: "Search failed. Please check your connection and try again.",
    funFactTitle: "Fun Fact",
    chat: "Chat",
    typeMessage: "Ask anything or say 'generate an image'...",
    aiGreeting: "Hi! Ask me anything about",
    generatingImg: "✨ Generating image based on: "
  },
  ru: {
    langName: "Russian",
    titleMode: "Точное название",
    vibeMode: "✨ ИИ Поиск по вайбу",
    placeholderTitle: "например: Начало, Во все тяжкие...",
    placeholderVibe: "например: Группа друзей теряется в жутком лесу...",
    searchBtn: "Поиск",
    suggestBtn: "✨ Найти",
    noResults: "Фильмы или сериалы не найдены.",
    tryVibe: "Попробуйте описать сюжет подробнее.",
    topMatches: "Лучшие совпадения",
    otherMedia: "Другие медиа (Игры, Эпизоды и др.)",
    info: "Инфо",
    aiPitch: "ИИ Описание",
    playMovie: "Смотреть",
    year: "Год",
    genre: "Жанр",
    director: "Режиссер / Разработчик",
    cast: "В главных ролях",
    unknown: "Неизвестно",
    gathering: "Сбор данных...",
    settings: "Настройки",
    language: "Язык",
    theme: "Тема",
    light: "Светлая",
    dark: "Темная",
    close: "Закрыть",
    aiPitchTitle: "✨ ИИ Описание и Факты",
    subtitle: "Ищите по точному названию или опишите атмосферу, и мы найдем прямую ссылку на трансляцию.",
    errorAI: "Не удалось сгенерировать ответ ИИ. Попробуйте еще раз.",
    errorInfo: "Не удалось загрузить информацию о фильме.",
    errorSearch: "Ошибка поиска. Проверьте подключение и повторите попытку.",
    funFactTitle: "Интересный факт",
    chat: "Чат",
    typeMessage: "Спросите меня о чем угодно или попросите картинку...",
    aiGreeting: "Привет! Спросите меня что-нибудь о",
    generatingImg: "✨ Генерирую изображение по запросу: "
  },
  az: {
    langName: "Azerbaijani",
    titleMode: "Dəqiq Ad",
    vibeMode: "✨ Sİ (Vibe) Axtarışı",
    placeholderTitle: "məs. Inception, Breaking Bad...",
    placeholderVibe: "məs. Bir qrup dost qorxulu meşədə azır...",
    searchBtn: "Axtar",
    suggestBtn: "✨ Təklif et",
    noResults: "Film və ya serial tapılmadı.",
    tryVibe: "Süjeti daha ətraflı təsvir etməyə çalışın.",
    topMatches: "Ən uyğun nəticələr",
    otherMedia: "Digər Mediya (Oyunlar, Epizodlar və s.)",
    info: "Məlumat",
    aiPitch: "Sİ Xülasəsi",
    playMovie: "İzlə",
    year: "İl",
    genre: "Janr",
    director: "Rejissor / Tərtibatçı",
    cast: "Əsas Rollarda",
    unknown: "Məlum deyil",
    gathering: "Məlumatlar toplanır...",
    settings: "Tənzimləmələr",
    language: "Dil",
    theme: "Mövzu",
    light: "Açıq",
    dark: "Tünd",
    close: "Bağla",
    aiPitchTitle: "✨ Sİ Xülasəsi və Faktlar",
    subtitle: "Dəqiq adla axtarın və ya axtardığınız ab-havanı təsvir edin, birbaşa izləmə linkini tapaq.",
    errorAI: "Sİ cavab verə bilmədi. Zəhmət olmasa yenidən cəhd edin.",
    errorInfo: "Film məlumatı yüklənə bilmədi.",
    errorSearch: "Axtarış alınmadı. İnternet bağlantınızı yoxlayın və yenidən cəhd edin.",
    funFactTitle: "Maraqlı Fakt",
    chat: "Söhbət",
    typeMessage: "Məndən hər şey soruşa və ya şəkil yarada bilərsiniz...",
    aiGreeting: "Salam! Məndən soruşun:",
    generatingImg: "✨ Təsvir yaradılır: "
  }
};

export default function App() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [searched, setSearched] = useState(false);
  const [lastSearchMode, setLastSearchMode] = useState('title');
  const [searchMode, setSearchMode] = useState('title');
  
  // Settings
  const [language, setLanguage] = useState('en');
  const [theme, setTheme] = useState('dark');
  const [showSettings, setShowSettings] = useState(false);

  // Per-item UI State
  const [expandedTab, setExpandedTab] = useState({});
  const [pitches, setPitches] = useState({});
  const [pitchLoading, setPitchLoading] = useState({});
  const [movieInfo, setMovieInfo] = useState({});
  const [infoLoading, setInfoLoading] = useState({});
  const [chatMessages, setChatMessages] = useState({});
  const [chatInput, setChatInput] = useState({});
  const [isChatLoading, setIsChatLoading] = useState({});
  
  const textareaRef = useRef(null);
  const chatRefs = useRef({});

  const t = (key) => locales[language][key] || locales['en'][key];
  const langName = locales[language].langName;

  // Sync scroll on query change
  useEffect(() => {
    if (textareaRef.current) {
      textareaRef.current.style.height = 'auto';
      textareaRef.current.style.height = `${textareaRef.current.scrollHeight}px`;
    }
  }, [query]);

  // Language Change Cleanup
  useEffect(() => {
    setResults([]);
    setSearched(false);
    setExpandedTab({});
    setPitches({});
    setMovieInfo({});
    setChatMessages({});
    setError(null);
  }, [language]);

  // Scroll individual chats
  useEffect(() => {
    Object.values(chatRefs.current).forEach(ref => {
      if (ref) ref.scrollIntoView({ behavior: 'smooth' });
    });
  }, [chatMessages, isChatLoading]);

  // --- Core API Wrapper ---
  const callGemini = async (prompt, isJson = false) => {
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
    let retries = 5;
    let delay = 1000;
    while (retries > 0) {
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ parts: [{ text: prompt }] }],
            generationConfig: isJson ? { responseMimeType: "application/json" } : {}
          })
        });
        if (!response.ok) throw new Error();
        const data = await response.json();
        const text = data.candidates?.[0]?.content?.parts?.[0]?.text;
        if (!text) throw new Error();
        return isJson ? JSON.parse(text) : text;
      } catch (err) {
        retries--;
        await new Promise(r => setTimeout(r, delay));
        delay *= 2;
      }
    }
    throw new Error("API Limit");
  };

  const safeRender = (val) => {
    if (val === undefined || val === null) return '';
    if (typeof val === 'object' && !Array.isArray(val)) return JSON.stringify(val);
    if (Array.isArray(val)) return val.join(', ');
    return String(val);
  };

  // --- Data Fetching ---
  const fetchWikidataForTitle = async (searchTitle) => {
    try {
      const searchRes = await fetch(
        `https://www.wikidata.org/w/api.php?action=wbsearchentities&search=${encodeURIComponent(searchTitle)}&language=${language}&format=json&origin=*`
      );
      const searchData = await searchRes.json();
      if (!searchData.search || searchData.search.length === 0) return [];
      const topResults = searchData.search.slice(0, 8);
      const ids = topResults.map(item => item.id).join('|');
      const entitiesRes = await fetch(
        `https://www.wikidata.org/w/api.php?action=wbgetentities&ids=${ids}&props=claims&format=json&origin=*`
      );
      const entitiesData = await entitiesRes.json();
      return topResults.map(item => {
        const entity = entitiesData.entities[item.id];
        const imdbId = entity?.claims?.P345?.[0]?.mainsnak?.datavalue?.value;
        const desc = item.description?.toLowerCase() || '';
        const isMain = /(film|movie|television|tv|series|miniseries|drama|sitcom|documentary|animation|anime|фильм|сериал|кино)/i.test(desc) 
          && !/(episode|character|video game|franchise|album|song|эпизод|персонаж|игра)/i.test(desc);
        return {
          id: item.id,
          title: item.label,
          description: item.description || t('noResults'),
          imdbId: imdbId,
          isMain: isMain
        };
      }).filter(item => item.imdbId);
    } catch (e) { return []; }
  };

  const handleSearch = async (e) => {
    e.preventDefault();
    if (!query.trim()) return;
    setLoading(true); setError(null); setSearched(true); setLastSearchMode(searchMode); setResults([]);
    try {
      let finalResults = [];
      if (searchMode === 'vibe') {
        const prompt = `User Query in ${langName}: "${query}". Suggest 3-4 famous ORIGINAL movie/TV show titles matching this description. If it's an exact title, put that first. Return JSON array of strings only.`;
        const suggested = await callGemini(prompt, true);
        const wikiPromises = suggested.map(title => fetchWikidataForTitle(title));
        const wikiResults = await Promise.all(wikiPromises);
        const seen = new Set();
        wikiResults.flat().forEach(item => { if (!seen.has(item.imdbId)) { seen.add(item.imdbId); finalResults.push(item); } });
      } else {
        const raw = await fetchWikidataForTitle(query);
        const seen = new Set();
        raw.forEach(item => { if (!seen.has(item.imdbId)) { seen.add(item.imdbId); finalResults.push(item); } });
      }
      setResults(finalResults);
    } catch (err) { setError(t('errorSearch')); }
    finally { setLoading(false); }
  };

  // --- Feature Handlers ---
  const fetchPitch = async (item) => {
    setPitchLoading(prev => ({ ...prev, [item.id]: true }));
    try {
      const prompt = ` critic pitch for "${item.title}" (${item.description}) in ${langName}. 2 sentences + fun fact line starting with "**${t('funFactTitle')}:** ". No hallucination.`;
      const text = await callGemini(prompt, false);
      setPitches(prev => ({ ...prev, [item.id]: text }));
    } catch (e) { setPitches(prev => ({ ...prev, [item.id]: t('errorAI') })); }
    finally { setPitchLoading(prev => ({ ...prev, [item.id]: false })); }
  };

  const fetchInfo = async (item) => {
    setInfoLoading(prev => ({ ...prev, [item.id]: true }));
    try {
      const prompt = `JSON facts for "${item.title}" (${item.description}) in ${langName}. Keys: year, director, cast, genre.`;
      const data = await callGemini(prompt, true);
      setMovieInfo(prev => ({ ...prev, [item.id]: data }));
    } catch (e) { setMovieInfo(prev => ({ ...prev, [item.id]: { error: true } })); }
    finally { setInfoLoading(prev => ({ ...prev, [item.id]: false })); }
  };

  const handleSendMessage = async (e, item) => {
    e.preventDefault();
    const userMsg = chatInput[item.id]?.trim();
    if (!userMsg || isChatLoading[item.id]) return;
    setChatInput(prev => ({ ...prev, [item.id]: '' }));
    setChatMessages(prev => ({ ...prev, [item.id]: [...(prev[item.id] || []), { role: 'user', text: userMsg }] }));
    setIsChatLoading(prev => ({ ...prev, [item.id]: true }));
    try {
      const history = (chatMessages[item.id] || []).map(m => `${m.role}: ${m.text}`).join('\n');
      const prompt = `Assistant about "${item.title}" (${item.description}) in ${langName}. If asked for image, return EXACTLY: [GENERATE_IMAGE: detailed prompt]. History: ${history} User: ${userMsg}`;
      let reply = await callGemini(prompt, false);
      const imgMatch = reply.match(/\[GENERATE_IMAGE:\s*(.*?)\]/i);
      if (imgMatch) {
          const imgP = imgMatch[1];
          setChatMessages(prev => ({ ...prev, [item.id]: [...(prev[item.id] || []), { role: 'model', text: `*${t('generatingImg')}"${imgP}"...*` }] }));
          const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${apiKey}`, {
              method: 'POST', headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ instances: { prompt: imgP }, parameters: { sampleCount: 1 } })
          });
          const d = await res.json();
          if (d.predictions?.[0]) reply = `![Generated Image](data:image/png;base64,${d.predictions[0].bytesBase64Encoded})`;
          else reply = "*(Image block/error)*";
          setChatMessages(prev => {
            const arr = [...(prev[item.id] || [])];
            arr[arr.length - 1] = { role: 'model', text: reply };
            return { ...prev, [item.id]: arr };
          });
          setIsChatLoading(prev => ({ ...prev, [item.id]: false }));
          return;
      }
      setChatMessages(prev => ({ ...prev, [item.id]: [...(prev[item.id] || []), { role: 'model', text: reply }] }));
    } catch (e) { setChatMessages(prev => ({ ...prev, [item.id]: [...(prev[item.id] || []), { role: 'model', text: t('errorAI') }] })); }
    finally { setIsChatLoading(prev => ({ ...prev, [item.id]: false })); }
  };

  const renderMarkdown = (text) => {
    if (!text || typeof text !== 'string') return null;
    return text.split('\n').filter(p => p.trim()).map((para, i) => {
      const img = para.match(/^!\[(.*?)\]\((.*?)\)$/);
      if (img) return <img key={i} src={img[2]} alt={img[1]} className="my-3 rounded-xl w-full max-w-sm border dark:border-slate-700 shadow-lg" />;
      let html = para.replace(/\*\*(.*?)\*\*/g, '<strong class="text-indigo-600 dark:text-indigo-400">$1</strong>')
                     .replace(/\*(.*?)\*/g, '<em class="italic text-slate-500">$1</em>');
      return <p key={i} dangerouslySetInnerHTML={{ __html: html }} className="mb-2 leading-relaxed" />;
    });
  };

  const mainResults = results.filter(r => r.isMain);
  const otherResults = results.filter(r => !r.isMain);

  const renderCard = (item, isTop = false) => {
    const active = expandedTab[item.id];
    const cardContent = (
      <div className="space-y-4">
        <div className="flex flex-col sm:flex-row justify-between gap-4">
          <div className="flex-1">
            <h3 className="text-xl font-bold dark:text-white">{item.title}</h3>
            <p className="text-sm text-slate-500 line-clamp-2">{item.description}</p>
            <span className="mt-2 inline-block text-[10px] font-mono px-2 py-0.5 bg-slate-100 dark:bg-slate-950 rounded border dark:border-slate-800">IMDb: {item.imdbId}</span>
          </div>
          {item.isMain && (
            <div className="relative group shrink-0">
              {isTop && <div className="absolute inset-0 rounded-xl bg-emerald-500/40 blur-xl animate-pulse" />}
              <a href={`https://www.playimdb.com/title/${item.imdbId}`} target="_blank" className={`relative flex items-center justify-center gap-2 px-6 py-2 rounded-xl font-bold transition-all ${isTop ? 'bg-emerald-500 text-slate-900 shadow-xl' : 'bg-slate-800 text-white hover:bg-emerald-500'}`}>
                <Play className="w-4 h-4 fill-current" /> {t('playMovie')}
              </a>
            </div>
          )}
        </div>
        <div className="flex gap-2 flex-wrap border-t dark:border-slate-800 pt-4">
          {[ ['pitch', Sparkles, t('aiPitch')], ['info', Info, t('info')], ['chat', MessageCircle, t('chat')] ].map(([id, Icon, label]) => (
            <button key={id} onClick={() => toggleTab(item, id)} className={`flex items-center gap-2 px-3 py-1.5 rounded-lg text-xs font-semibold transition-all ${active === id ? 'bg-indigo-500 text-white' : 'bg-slate-100 dark:bg-slate-800 text-slate-500 hover:bg-slate-200'}`}>
              <Icon className="w-3.5 h-3.5" /> {label}
            </button>
          ))}
        </div>
        {active && (
          <div className="p-4 bg-slate-50 dark:bg-slate-950/50 rounded-xl border dark:border-slate-800">
            {active === 'pitch' && (pitchLoading[item.id] ? <Loader2 className="w-5 h-5 animate-spin mx-auto text-indigo-500" /> : renderMarkdown(pitches[item.id]))}
            {active === 'info' && (infoLoading[item.id] ? <Loader2 className="w-5 h-5 animate-spin mx-auto text-indigo-500" /> : (
              <div className="grid grid-cols-2 gap-4 text-xs">
                {Object.entries({ [t('year')]: movieInfo[item.id]?.year, [t('genre')]: movieInfo[item.id]?.genre, [t('director')]: movieInfo[item.id]?.director, [t('cast')]: movieInfo[item.id]?.cast }).map(([k,v]) => (
                  <div key={k}><span className="block text-[10px] uppercase text-slate-400">{k}</span><span className="font-bold dark:text-slate-200">{safeRender(v) || t('unknown')}</span></div>
                ))}
              </div>
            ))}
            {active === 'chat' && (
              <div className="flex flex-col h-[350px]">
                <div className="flex-1 overflow-y-auto space-y-3 pr-2 mb-4">
                  {(chatMessages[item.id] || []).map((m,i) => (
                    <div key={i} className={`flex ${m.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                      <div className={`px-3 py-2 rounded-2xl text-xs max-w-[85%] ${m.role === 'user' ? 'bg-indigo-600 text-white rounded-tr-none' : 'bg-white dark:bg-slate-800 border dark:border-slate-700 rounded-tl-none'}`}>
                        {m.role === 'model' ? renderMarkdown(m.text) : m.text}
                      </div>
                    </div>
                  ))}
                  {isChatLoading[item.id] && <Loader2 className="w-4 h-4 animate-spin text-indigo-500" />}
                  <div ref={el => chatRefs.current[item.id] = el} />
                </div>
                <form onSubmit={e => handleSendMessage(e, item)} className="relative flex gap-2">
                  <input type="text" value={chatInput[item.id] || ''} onChange={e => setChatInput(prev => ({...prev, [item.id]: e.target.value}))} placeholder={t('typeMessage')} className="w-full bg-white dark:bg-slate-900 border dark:border-slate-800 rounded-lg px-4 py-2 text-xs focus:ring-1 ring-indigo-500 outline-none" />
                  <button type="submit" disabled={isChatLoading[item.id] || !chatInput[item.id]?.trim()} className="p-2 bg-indigo-600 text-white rounded-lg disabled:opacity-50"><Send className="w-4 h-4"/></button>
                </form>
              </div>
            )}
          </div>
        )}
      </div>
    );

    const toggleTab = (it, tb) => {
        if(expandedTab[it.id] === tb) { setExpandedTab(p => ({...p, [it.id]: null})); return; }
        setExpandedTab(p => ({...p, [it.id]: tb}));
        if(tb === 'pitch' && !pitches[it.id]) fetchPitch(it);
        if(tb === 'info' && !movieInfo[it.id]) fetchInfo(it);
        if(tb === 'chat' && !(chatMessages[it.id]?.length)) setChatMessages(p => ({...p, [it.id]: [{role: 'model', text: `${t('aiGreeting')} **${it.title}**!`}]}));
    }

    return (
      <div key={item.id} className={`relative p-0.5 rounded-2xl transition-all mb-4 ${isTop ? 'bg-gradient-to-r from-amber-400 via-emerald-500 to-amber-400 shadow-[0_0_20px_rgba(245,158,11,0.3)] animate-pulse' : ''}`}>
        <div className={`p-5 rounded-[14px] bg-white dark:bg-slate-900 border dark:border-slate-800 shadow-sm ${isTop ? 'border-none' : ''}`}>
          {cardContent}
        </div>
      </div>
    );
  };

  return (
    <div className={`${theme} min-h-screen transition-colors duration-300 font-sans`}>
      <div className="bg-slate-50 dark:bg-slate-950 min-h-screen text-slate-800 dark:text-slate-200 p-4 sm:p-8">
        <div className="max-w-3xl mx-auto relative">
          <button onClick={() => setShowSettings(true)} className="absolute top-0 right-0 p-2 bg-white dark:bg-slate-900 border dark:border-slate-800 rounded-full shadow hover:scale-110 transition-transform"><Settings className="w-5 h-5"/></button>
          <div className="text-center pt-12 mb-8 space-y-4">
            <div className="inline-block p-4 bg-emerald-500/10 rounded-3xl relative"><Film className="w-12 h-12 text-emerald-500"/><Sparkles className="absolute -top-1 -right-1 w-6 h-6 text-amber-400 animate-bounce"/></div>
            <h1 className="text-5xl font-black tracking-tighter dark:text-white uppercase">Play<span className="text-emerald-500">IMDB</span></h1>
            <p className="text-slate-500 max-w-sm mx-auto font-medium">{t('subtitle')}</p>
          </div>
          <div className="flex justify-center gap-2 mb-6">
            {['title', 'vibe'].map(m => (
              <button key={m} onClick={() => setSearchMode(m)} className={`px-5 py-2 rounded-full text-xs font-bold transition-all ${searchMode === m ? 'bg-slate-800 text-white dark:bg-white dark:text-slate-900' : 'text-slate-400'}`}>
                {m === 'title' ? t('titleMode') : t('vibeMode')}
              </button>
            ))}
          </div>
          <form onSubmit={handleSearch} className="relative mb-12">
            <div className="absolute left-4 top-5 text-slate-400">{searchMode === 'title' ? <Search size={20}/> : <Wand2 size={20}/>}</div>
            <textarea ref={textareaRef} rows={1} value={query} onChange={e => setQuery(e.target.value)} onKeyDown={e => e.key === 'Enter' && !e.shiftKey && handleSearch(e)} className="w-full bg-white dark:bg-slate-900 border-2 dark:border-slate-800 rounded-3xl pl-12 pr-32 py-4 text-lg focus:border-emerald-500 outline-none transition-all shadow-xl resize-none overflow-hidden min-h-[64px]" placeholder={searchMode === 'title' ? t('placeholderTitle') : t('placeholderVibe')} />
            <button type="submit" disabled={loading || !query.trim()} className="absolute right-2 top-2 bottom-2 bg-emerald-500 text-slate-900 px-6 rounded-2xl font-black text-sm hover:bg-emerald-400 transition-colors uppercase tracking-widest disabled:opacity-50">
              {loading ? <Loader2 className="animate-spin w-5 h-5"/> : (searchMode === 'title' ? t('searchBtn') : t('suggestBtn'))}
            </button>
          </form>
          {error && <div className="p-4 bg-red-500/10 border border-red-500/30 rounded-2xl text-red-500 text-center mb-6">{error}</div>}
          <div className="space-y-8">
            {mainResults.length > 0 && <div><h2 className="text-xs font-black uppercase tracking-widest text-slate-400 mb-4 px-2">{t('topMatches')}</h2>{mainResults.map((r,i) => renderCard(r, lastSearchMode === 'vibe' && i === 0))}</div>}
            {otherResults.length > 0 && <div className="pt-8 border-t dark:border-slate-900"><h2 className="text-xs font-black uppercase tracking-widest text-slate-400 mb-4 px-2">{t('otherMedia')}</h2>{otherResults.map(r => renderCard(r))}</div>}
            {searched && !loading && !results.length && <div className="text-center py-20 opacity-30 font-bold text-2xl">{t('noResults')}</div>}
          </div>
        </div>
      </div>
      {showSettings && (
        <div className="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-slate-950/80 backdrop-blur-md">
          <div className="bg-white dark:bg-slate-900 w-full max-w-sm rounded-[40px] p-8 relative shadow-2xl border dark:border-slate-800">
            <button onClick={() => setShowSettings(false)} className="absolute top-8 right-8 text-slate-400 hover:text-white"><X/></button>
            <h2 className="text-3xl font-black mb-8 dark:text-white">{t('settings')}</h2>
            <div className="space-y-6">
              <div><span className="block text-[10px] font-black uppercase text-slate-400 mb-2 tracking-widest">{t('language')}</span>
                <div className="grid grid-cols-1 gap-2">{Object.keys(locales).map(l => (
                  <button key={l} onClick={() => setLanguage(l)} className={`p-4 rounded-2xl border-2 text-left font-bold transition-all ${language === l ? 'border-emerald-500 bg-emerald-500/10 text-emerald-500' : 'border-slate-100 dark:border-slate-800 text-slate-400'}`}>{locales[l].langName}</button>
                ))}</div>
              </div>
              <div><span className="block text-[10px] font-black uppercase text-slate-400 mb-2 tracking-widest">{t('theme')}</span>
                <div className="grid grid-cols-2 gap-2">{['light', 'dark'].map(m => (
                  <button key={m} onClick={() => setTheme(m)} className={`p-4 rounded-2xl border-2 font-bold flex items-center justify-center gap-2 ${theme === m ? 'border-emerald-500 bg-emerald-500/10 text-emerald-500' : 'border-slate-100 dark:border-slate-800 text-slate-400'}`}>{m === 'light' ? <Sun size={18}/> : <Moon size={18}/>}{t(m)}</button>
                ))}</div>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}