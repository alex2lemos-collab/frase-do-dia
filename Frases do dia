import React, { useState, useEffect, useRef } from 'react';
import { 
  Heart, Share2, RefreshCw, Calendar, Sparkles, Quote, 
  Trash2, BrainCircuit, Volume2, Wand2, Loader2 
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';

// --- CONFIGURAÇÕES ---
const apiKey = ""; // Chave da API fornecida pelo ambiente

const quotesData = [
  { id: 1, text: "A única maneira de fazer um excelente trabalho é amar o que você faz.", author: "Steve Jobs", category: "Trabalho" },
  { id: 2, text: "O sucesso é a soma de pequenos esforços repetidos dia após dia.", author: "Robert Collier", category: "Persistência" },
  { id: 3, text: "Acredite que você pode, assim você já está no meio do caminho.", author: "Theodore Roosevelt", category: "Confiança" },
  { id: 4, text: "O futuro pertence àqueles que acreditam na beleza de seus sonhos.", author: "Eleanor Roosevelt", category: "Sonhos" },
  { id: 5, text: "Comece onde você está. Use o que você tem. Faça o que você pode.", author: "Arthur Ashe", category: "Ação" },
  { id: 6, text: "Imagine uma nova história para sua vida e acredite nela.", author: "Paulo Coelho", category: "Vida" },
  { id: 7, text: "A vida é 10% o que acontece com você e 90% como você reage a isso.", author: "Charles R. Swindoll", category: "Atitude" },
  { id: 8, text: "Não importa o quão devagar você vá, desde que você não pare.", author: "Confúcio", category: "Progresso" }
];

// --- COMPONENTE DE ANÚNCIO ---
const AdBanner = ({ type = 'standard' }) => (
  <div className={`my-6 mx-auto bg-slate-100 border-2 border-dashed border-slate-300 flex flex-col items-center justify-center text-slate-400 rounded-lg overflow-hidden relative group cursor-pointer hover:bg-slate-50 transition-colors ${
    type === 'large' ? 'w-[300px] h-[250px]' : 'w-[320px] h-[50px]'
  }`}>
    <span className="text-[10px] font-bold uppercase tracking-widest mb-1">Publicidade</span>
    <div className="absolute inset-0 bg-indigo-600/5 opacity-0 group-hover:opacity-100 transition-opacity"></div>
  </div>
);

export default function App() {
  const [currentView, setCurrentView] = useState('daily');
  const [currentQuote, setCurrentQuote] = useState(null);
  const [favorites, setFavorites] = useState([]);
  const [notification, setNotification] = useState(null);
  
  // Estados para as funções Gemini
  const [aiReflection, setAiReflection] = useState("");
  const [isLoadingAi, setIsLoadingAi] = useState(false);
  const [isSpeaking, setIsSpeaking] = useState(false);
  const audioContextRef = useRef(null);

  useEffect(() => {
    const savedFavorites = localStorage.getItem('motivationalFavorites');
    if (savedFavorites) setFavorites(JSON.parse(savedFavorites));
    setDailyQuote();
  }, []);

  const setDailyQuote = () => {
    const today = new Date();
    const seed = today.getFullYear() * 1000 + (today.getMonth() + 1) * 100 + today.getDate();
    const index = seed % quotesData.length;
    setCurrentQuote(quotesData[index]);
    setAiReflection("");
    setCurrentView('daily');
  };

  const setRandomQuote = () => {
    let randomIndex;
    do {
      randomIndex = Math.floor(Math.random() * quotesData.length);
    } while (currentQuote && quotesData[randomIndex].id === currentQuote.id);
    setCurrentQuote(quotesData[randomIndex]);
    setAiReflection("");
    setCurrentView('explore');
  };

  // --- INTEGRAÇÃO GEMINI: REFLEXÃO ---
  const generateAiReflection = async () => {
    if (!currentQuote || isLoadingAi) return;
    setIsLoadingAi(true);
    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: `Explique brevemente (máximo 2 frases) como posso aplicar esta frase na minha vida hoje: "${currentQuote.text}"` }] }],
          systemInstruction: { parts: [{ text: "Você é um mentor de vida motivacional gentil e prático." }] }
        })
      });
      const data = await response.json();
      const text = data.candidates?.[0]?.content?.parts?.[0]?.text;
      setAiReflection(text);
    } catch (error) {
      showNotification("Erro ao conectar com a IA");
    } finally {
      setIsLoadingAi(false);
    }
  };

  // --- INTEGRAÇÃO GEMINI: GERAR FRASE MÁGICA ---
  const generateMagicQuote = async () => {
    setIsLoadingAi(true);
    setAiReflection("");
    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: "Gere uma frase motivacional única e curta." }] }],
          generationConfig: {
            responseMimeType: "application/json",
            responseSchema: {
              type: "OBJECT",
              properties: {
                text: { type: "STRING" },
                author: { type: "STRING" }
              }
            }
          }
        })
      });
      const data = await response.json();
      const result = JSON.parse(data.candidates[0].content.parts[0].text);
      setCurrentQuote({ id: Date.now(), text: result.text, author: result.author, category: "Mágica ✨" });
      setCurrentView('explore');
    } catch (error) {
      showNotification("A magia falhou, tente novamente");
    } finally {
      setIsLoadingAi(false);
    }
  };

  // --- INTEGRAÇÃO GEMINI: TEXT-TO-SPEECH (OUVIR) ---
  const playQuoteAudio = async () => {
    if (isSpeaking) return;
    setIsSpeaking(true);
    try {
      const textToSay = `Reflexão de hoje: ${currentQuote.text}. Por ${currentQuote.author}`;
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          model: "gemini-2.5-flash-preview-tts",
          contents: [{ parts: [{ text: `Diga com uma voz inspiradora e calma: ${textToSay}` }] }],
          generationConfig: {
            responseModalities: ["AUDIO"],
            speechConfig: { voiceConfig: { prebuiltVoiceConfig: { voiceName: "Puck" } } }
          }
        })
      });
      
      const data = await response.json();
      const audioData = data.candidates[0].content.parts[0].inlineData.data;
      
      // Converte PCM para WAV para tocar no navegador
      const audioBlob = pcmToWav(audioData, 24000); // Exemplo de sample rate
      const url = URL.createObjectURL(audioBlob);
      const audio = new Audio(url);
      audio.onended = () => setIsSpeaking(false);
      audio.play();
    } catch (error) {
      showNotification("Erro ao gerar áudio");
      setIsSpeaking(false);
    }
  };

  // Helper simples para PCM para WAV (simplificado)
  const pcmToWav = (base64Pcm, sampleRate) => {
    const byteCharacters = atob(base64Pcm);
    const byteNumbers = new Array(byteCharacters.length);
    for (let i = 0; i < byteCharacters.length; i++) {
      byteNumbers[i] = byteCharacters.charCodeAt(i);
    }
    const byteArray = new Uint8Array(byteNumbers);
    const buffer = byteArray.buffer;
    
    // Header WAV básico (44 bytes)
    const header = new ArrayBuffer(44);
    const view = new DataView(header);
    const writeString = (offset, string) => {
      for (let i = 0; i < string.length; i++) view.setUint8(offset + i, string.charCodeAt(i));
    };
    writeString(0, 'RIFF');
    view.setUint32(4, 36 + buffer.byteLength, true);
    writeString(8, 'WAVE');
    writeString(12, 'fmt ');
    view.setUint32(16, 16, true);
    view.setUint16(20, 1, true); // Linear PCM
    view.setUint16(22, 1, true); // Mono
    view.setUint32(24, sampleRate, true);
    view.setUint32(28, sampleRate * 2, true);
    view.setUint16(32, 2, true);
    view.setUint16(34, 16, true);
    writeString(36, 'data');
    view.setUint32(40, buffer.byteLength, true);

    return new Blob([header, buffer], { type: 'audio/wav' });
  };

  const toggleFavorite = (quote) => {
    let newFavs = favorites.some(f => f.id === quote.id)
      ? favorites.filter(f => f.id !== quote.id)
      : [...favorites, quote];
    setFavorites(newFavs);
    localStorage.setItem('motivationalFavorites', JSON.stringify(newFavs));
    showNotification(favorites.some(f => f.id === quote.id) ? "Removido" : "Favoritado ❤️");
  };

  const copyToClipboard = (text, author) => {
    const full = `"${text}" - ${author}`;
    navigator.clipboard.writeText(full).then(() => showNotification("Copiado!"));
  };

  const showNotification = (msg) => {
    setNotification(msg);
    setTimeout(() => setNotification(null), 2500);
  };

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800 font-sans flex flex-col max-w-md mx-auto shadow-2xl overflow-hidden relative border-x border-slate-200">
      
      {/* HEADER */}
      <header className="bg-white/90 backdrop-blur-md sticky top-0 z-30 px-6 py-4 flex items-center justify-between border-b border-slate-100 shadow-sm">
        <div className="flex items-center gap-2">
          <div className="bg-gradient-to-br from-indigo-600 to-purple-600 text-white p-1.5 rounded-lg shadow-md">
            <Sparkles size={18} fill="currentColor" />
          </div>
          <h1 className="text-xl font-bold bg-gradient-to-r from-indigo-600 to-purple-600 bg-clip-text text-transparent">
            MindfulDay
          </h1>
        </div>
      </header>

      <main className="flex-1 flex flex-col overflow-y-auto pb-24 relative bg-[radial-gradient(#e2e8f0_1px,transparent_1px)] [background-size:20px_20px]">
        
        {notification && (
          <div className="fixed top-20 left-1/2 -translate-x-1/2 bg-slate-800 text-white px-5 py-2 rounded-full text-xs shadow-xl z-50 animate-bounce">
            {notification}
          </div>
        )}

        {/* VIEW: FAVORITOS */}
        {currentView === 'favorites' && (
          <div className="p-6 space-y-4 animate-fade-in">
            <h2 className="text-xl font-bold flex items-center gap-2">Meus Favoritos ({favorites.length})</h2>
            {favorites.length === 0 ? (
              <p className="text-center py-20 text-slate-400">Nenhuma frase guardada ainda.</p>
            ) : (
              favorites.map(f => (
                <div key={f.id} className="bg-white p-4 rounded-xl border border-slate-100 shadow-sm relative group">
                  <p className="text-sm font-medium mb-2">"{f.text}"</p>
                  <p className="text-xs text-slate-400 italic">— {f.author}</p>
                  <button onClick={() => toggleFavorite(f)} className="absolute top-2 right-2 text-slate-200 hover:text-red-500"><Trash2 size={16} /></button>
                </div>
              ))
            )}
            <AdBanner type="large" />
          </div>
        )}

        {/* VIEW: PRINCIPAL */}
        {(currentView === 'daily' || currentView === 'explore') && currentQuote && (
          <div className="flex-1 flex flex-col items-center justify-center px-8 pt-8 animate-fade-in">
            
            <div className="mb-6 flex gap-2">
              <span className="px-3 py-1 bg-indigo-50 text-indigo-600 text-[10px] font-bold rounded-full border border-indigo-100 uppercase">
                {currentQuote.category}
              </span>
            </div>

            <div className="bg-white p-8 rounded-3xl shadow-xl shadow-indigo-100/50 border border-slate-100 w-full relative group">
              <Quote size={32} className="text-indigo-100 mb-4" />
              <h2 className="text-2xl font-bold text-center leading-tight mb-6">
                {currentQuote.text}
              </h2>
              <p className="text-center text-slate-400 font-serif">— {currentQuote.author}</p>
              
              {/* Botão Ouvir IA */}
              <button 
                onClick={playQuoteAudio}
                disabled={isSpeaking}
                className="absolute top-4 right-4 p-2 text-slate-300 hover:text-indigo-500 transition-colors"
              >
                {isSpeaking ? <Loader2 className="animate-spin" size={18} /> : <Volume2 size={18} />}
              </button>
            </div>

            {/* BOTÕES DE IA ✨ */}
            <div className="grid grid-cols-2 gap-3 w-full mt-6">
              <button 
                onClick={generateAiReflection}
                disabled={isLoadingAi}
                className="flex items-center justify-center gap-2 py-3 px-4 bg-white border border-indigo-100 rounded-2xl text-xs font-bold text-indigo-600 hover:bg-indigo-50 transition-all shadow-sm"
              >
                {isLoadingAi ? <Loader2 className="animate-spin" size={16} /> : <BrainCircuit size={16} />}
                ✨ Refletir com IA
              </button>
              <button 
                onClick={generateMagicQuote}
                disabled={isLoadingAi}
                className="flex items-center justify-center gap-2 py-3 px-4 bg-indigo-600 rounded-2xl text-xs font-bold text-white hover:bg-indigo-700 transition-all shadow-md"
              >
                {isLoadingAi ? <Loader2 className="animate-spin" size={16} /> : <Wand2 size={16} />}
                ✨ Gerar Frase IA
              </button>
            </div>

            {/* BOX DE REFLEXÃO DA IA */}
            {aiReflection && (
              <div className="mt-4 p-4 bg-gradient-to-br from-indigo-50 to-purple-50 rounded-2xl border border-indigo-100 animate-fade-in w-full">
                <p className="text-xs text-indigo-800 leading-relaxed italic">
                  <span className="font-bold block mb-1">Dica da IA:</span>
                  {aiReflection}
                </p>
              </div>
            )}

            {/* AÇÕES BÁSICAS */}
            <div className="flex gap-4 mt-8">
              <button onClick={() => toggleFavorite(currentQuote)} className={`p-4 rounded-full shadow-lg ${favorites.some(f => f.id === currentQuote.id) ? 'bg-red-500 text-white' : 'bg-white text-slate-400'}`}>
                <Heart size={20} fill={favorites.some(f => f.id === currentQuote.id) ? "currentColor" : "none"} />
              </button>
              <button onClick={setRandomQuote} className="p-4 bg-white text-slate-400 rounded-full shadow-lg">
                <RefreshCw size={20} />
              </button>
              <button onClick={() => copyToClipboard(currentQuote.text, currentQuote.author)} className="p-4 bg-white text-slate-400 rounded-full shadow-lg">
                <Share2 size={20} />
              </button>
            </div>

            <AdBanner />
          </div>
        )}
      </main>

      {/* NAV BAR */}
      <nav className="bg-white/80 backdrop-blur-md border-t border-slate-100 flex justify-around items-center py-4 absolute bottom-0 w-full z-40">
        <button onClick={setDailyQuote} className={`flex flex-col items-center gap-1 ${currentView === 'daily' ? 'text-indigo-600' : 'text-slate-400'}`}>
          <Calendar size={20} />
          <span className="text-[10px] font-bold">Hoje</span>
        </button>
        <button onClick={() => setCurrentView('explore')} className={`flex flex-col items-center gap-1 ${currentView === 'explore' ? 'text-indigo-600' : 'text-slate-400'}`}>
          <RefreshCw size={20} />
          <span className="text-[10px] font-bold">Explorar</span>
        </button>
        <button onClick={() => setCurrentView('favorites')} className={`flex flex-col items-center gap-1 ${currentView === 'favorites' ? 'text-indigo-600' : 'text-slate-400'}`}>
          <Heart size={20} />
          <span className="text-[10px] font-bold">Favoritos</span>
        </button>
      </nav>

      <style>{`
        @keyframes fade-in { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-fade-in { animation: fade-in 0.3s ease-out; }
      `}</style>
    </div>
  );
}
