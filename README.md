import React, { useState, useEffect } from 'react';
import { Heart, Share2, RefreshCw, Calendar, Sparkles, Quote, Menu, X, Trash2, Info } from 'lucide-react';

// --- DADOS ---
const quotesData = [
  { id: 1, text: "A única maneira de fazer um excelente trabalho é amar o que você faz.", author: "Steve Jobs", category: "Trabalho" },
  { id: 2, text: "O sucesso é a soma de pequenos esforços repetidos dia após dia.", author: "Robert Collier", category: "Persistência" },
  { id: 3, text: "Não espere por oportunidades extraordinárias. Agarre ocasiões comuns e torne-as grandes.", author: "Orison Swett Marden", category: "Ação" },
  { id: 4, text: "Acredite que você pode, assim você já está no meio do caminho.", author: "Theodore Roosevelt", category: "Confiança" },
  { id: 5, text: "O futuro pertence àqueles que acreditam na beleza de seus sonhos.", author: "Eleanor Roosevelt", category: "Sonhos" },
  { id: 6, text: "Comece onde você está. Use o que você tem. Faça o que você pode.", author: "Arthur Ashe", category: "Ação" },
  { id: 7, text: "Tudo o que um sonho precisa para ser realizado é alguém que acredite que ele possa ser realizado.", author: "Roberto Shinyashiki", category: "Sonhos" },
  { id: 8, text: "Imagine uma nova história para sua vida e acredite nela.", author: "Paulo Coelho", category: "Vida" },
  { id: 9, text: "O insucesso é apenas uma oportunidade para recomeçar com mais inteligência.", author: "Henry Ford", category: "Resiliência" },
  { id: 10, text: "Você nunca sabe que resultados virão da sua ação. Mas se você não fizer nada, não existirão resultados.", author: "Mahatma Gandhi", category: "Ação" },
  { id: 11, text: "Persistência é a irmã gêmea da excelência. Uma é mãe da qualidade, a outra é a mãe do tempo.", author: "Marabel Morgan", category: "Persistência" },
  { id: 12, text: "A alegria de fazer o bem é a única felicidade verdadeira.", author: "Leon Tolstói", category: "Felicidade" },
  { id: 13, text: "Não deixe que o ruído da opinião alheia impeça que você escute a sua voz interior.", author: "Steve Jobs", category: "Autoconfiança" },
  { id: 14, text: "A vida é 10% o que acontece com você e 90% como você reage a isso.", author: "Charles R. Swindoll", category: "Atitude" },
  { id: 15, text: "A coragem não é ausência do medo; é a persistência apesar do medo.", author: "Desconhecido", category: "Coragem" },
  { id: 16, text: "Só se vê bem com o coração, o essencial é invisível aos olhos.", author: "Antoine de Saint-Exupéry", category: "Sabedoria" },
  { id: 17, text: "Transportai um punhado de terra todos os dias e fareis uma montanha.", author: "Confúcio", category: "Persistência" },
  { id: 18, text: "Se você quer viver uma vida feliz, amarre-se a uma meta, não às pessoas nem às coisas.", author: "Albert Einstein", category: "Felicidade" },
  { id: 19, text: "O melhor momento para plantar uma árvore foi há 20 anos. O segundo melhor momento é agora.", author: "Provérbio Chinês", category: "Tempo" },
  { id: 20, text: "Não importa o quão devagar você vá, desde que você não pare.", author: "Confúcio", category: "Progresso" },
  { id: 21, text: "A disciplina é a ponte entre metas e realizações.", author: "Jim Rohn", category: "Disciplina" },
  { id: 22, text: "Sorte é o que acontece quando a preparação encontra a oportunidade.", author: "Sêneca", category: "Sorte" },
  { id: 23, text: "Você não pode mudar o vento, mas pode ajustar as velas do barco para chegar onde quer.", author: "Confúcio", category: "Adaptação" },
  { id: 24, text: "A gratidão transforma o que temos em suficiente.", author: "Melody Beattie", category: "Gratidão" },
  { id: 25, text: "Faça o que for necessário para ser feliz. Mas não se esqueça que a felicidade é um sentimento simples, você pode encontrá-la e deixá-la ir embora por não perceber sua simplicidade.", author: "Mario Quintana", category: "Felicidade" }
];

// --- COMPONENTE DE ANÚNCIO (Placeholder) ---
const AdBanner = ({ type = 'standard' }) => {
  // Simula um espaço publicitário (AdSense/AdMob)
  // No futuro, você substituiria o conteúdo desta div pelo script do Google Ads
  return (
    <div className={`my-6 mx-auto bg-slate-100 border-2 border-dashed border-slate-300 flex flex-col items-center justify-center text-slate-400 rounded-lg overflow-hidden relative group cursor-pointer hover:bg-slate-50 transition-colors ${
      type === 'large' ? 'w-[300px] h-[250px]' : 'w-[320px] h-[50px]'
    }`}>
      <span className="text-xs font-bold uppercase tracking-widest mb-1">Publicidade</span>
      <span className="text-[10px] opacity-70">
        {type === 'large' ? 'Retângulo Médio (300x250)' : 'Banner Mobile (320x50)'}
      </span>
      
      {/* Overlay explicativo (apenas para visualização do dev) */}
      <div className="absolute inset-0 bg-indigo-600/10 opacity-0 group-hover:opacity-100 flex items-center justify-center transition-opacity">
        <span className="text-indigo-600 font-bold text-xs bg-white px-2 py-1 rounded shadow">Ad Space</span>
      </div>
    </div>
  );
};

export default function App() {
  const [currentView, setCurrentView] = useState('daily');
  const [currentQuote, setCurrentQuote] = useState(null);
  const [favorites, setFavorites] = useState([]);
  const [notification, setNotification] = useState(null);

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
    setCurrentView('daily');
  };

  const setRandomQuote = () => {
    let randomIndex;
    do {
      randomIndex = Math.floor(Math.random() * quotesData.length);
    } while (currentQuote && quotesData[randomIndex].id === currentQuote.id);
    setCurrentQuote(quotesData[randomIndex]);
    setCurrentView('explore');
  };

  const toggleFavorite = (quote) => {
    let newFavorites;
    const isFav = favorites.some(fav => fav.id === quote.id);

    if (isFav) {
      newFavorites = favorites.filter(fav => fav.id !== quote.id);
      showNotification("Removido dos favoritos");
    } else {
      newFavorites = [...favorites, quote];
      showNotification("Adicionado aos favoritos ❤️");
    }

    setFavorites(newFavorites);
    localStorage.setItem('motivationalFavorites', JSON.stringify(newFavorites));
  };

  const copyToClipboard = (text, author) => {
    const fullText = `"${text}" - ${author}`;
    // Fallback simples para execCommand se clipboard API falhar (comum em iframes)
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(fullText).then(() => showNotification("Copiado!"));
    } else {
        // Método legado para garantir funcionamento
        const textArea = document.createElement("textarea");
        textArea.value = fullText;
        document.body.appendChild(textArea);
        textArea.select();
        document.execCommand("copy");
        document.body.removeChild(textArea);
        showNotification("Copiado!");
    }
  };

  const showNotification = (msg) => {
    setNotification(msg);
    setTimeout(() => setNotification(null), 2500);
  };

  const NavButton = ({ view, icon: Icon, label, onClick }) => (
    <button
      onClick={onClick}
      className={`flex flex-col items-center justify-center w-full py-3 transition-colors ${
        currentView === view ? 'text-indigo-600' : 'text-slate-400 hover:text-slate-600'
      }`}
    >
      <Icon size={24} strokeWidth={currentView === view ? 2.5 : 2} />
      <span className="text-xs mt-1 font-medium">{label}</span>
    </button>
  );

  return (
    <div className="min-h-screen bg-slate-50 text-slate-800 font-sans flex flex-col max-w-md mx-auto shadow-2xl overflow-hidden relative">
      
      {/* Header */}
      <header className="bg-white/90 backdrop-blur-md sticky top-0 z-10 px-6 py-4 flex items-center justify-between border-b border-slate-100 shadow-sm">
        <div className="flex items-center gap-2">
          <div className="bg-gradient-to-br from-indigo-600 to-purple-600 text-white p-1.5 rounded-lg shadow-md shadow-indigo-200">
            <Sparkles size={18} fill="currentColor" />
          </div>
          <h1 className="text-xl font-bold bg-gradient-to-r from-indigo-600 to-purple-600 bg-clip-text text-transparent">
            MindfulDay
          </h1>
        </div>
      </header>

      {/* Conteúdo Principal */}
      <main className="flex-1 flex flex-col relative overflow-y-auto pb-20 scroll-smooth">
        
        {/* Notificação Toast */}
        {notification && (
          <div className="fixed top-20 left-1/2 transform -translate-x-1/2 bg-slate-800 text-white px-5 py-2.5 rounded-full text-sm shadow-xl z-50 animate-fade-in-down flex items-center gap-2 border border-slate-700">
            <Sparkles size={14} className="text-yellow-400" />
            {notification}
          </div>
        )}

        {/* --- VIEW: FAVORITOS --- */}
        {currentView === 'favorites' && (
          <div className="p-6 space-y-4 animate-fade-in pb-24">
            <h2 className="text-2xl font-bold text-slate-800 mb-6 flex items-center gap-2">
              Meus Favoritos <span className="text-sm font-normal text-slate-400">({favorites.length})</span>
            </h2>
            
            {favorites.length === 0 ? (
              <div className="text-center py-20 opacity-50">
                <Heart size={48} className="mx-auto mb-4 text-slate-300" />
                <p>Você ainda não tem favoritos.</p>
                <button 
                  onClick={() => {setRandomQuote(); setCurrentView('explore');}}
                  className="mt-4 text-indigo-600 font-semibold hover:underline"
                >
                  Explorar frases
                </button>
              </div>
            ) : (
              favorites.map((fav) => (
                <div key={fav.id} className="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 relative group transition-all hover:shadow-md">
                  <p className="text-slate-700 font-medium leading-relaxed mb-3 pr-8">"{fav.text}"</p>
                  <p className="text-sm text-slate-400 font-serif italic">— {fav.author}</p>
                  <div className="absolute top-4 right-4 flex flex-col gap-2">
                    <button onClick={() => toggleFavorite(fav)} className="text-slate-300 hover:text-red-500 transition-colors p-1"><Trash2 size={16} /></button>
                    <button onClick={() => copyToClipboard(fav.text, fav.author)} className="text-slate-300 hover:text-indigo-600 transition-colors p-1"><Share2 size={16} /></button>
                  </div>
                </div>
              ))
            )}
            
            {/* Anúncio no final da lista de favoritos */}
            {favorites.length > 0 && (
                <div className="pt-4 border-t border-slate-100 mt-8">
                    <p className="text-center text-xs text-slate-300 mb-2">Publicidade</p>
                    <AdBanner type="large" />
                </div>
            )}
          </div>
        )}

        {/* --- VIEW: DAILY & EXPLORE --- */}
        {(currentView === 'daily' || currentView === 'explore') && currentQuote && (
          <div className="flex-1 flex flex-col items-center pt-8 px-6 animate-fade-in pb-24">
            
            {/* Categoria Badge */}
            <span className="mb-8 px-3 py-1 bg-indigo-50 text-indigo-600 text-xs font-bold rounded-full uppercase tracking-wider border border-indigo-100">
              {currentView === 'daily' ? 'Frase do Dia' : currentQuote.category}
            </span>

            {/* Cartão da Frase Principal */}
            <div className="w-full relative max-w-sm">
              <Quote size={48} className="absolute -top-8 -left-2 text-indigo-100 transform -scale-x-100 opacity-80" />
              
              <h2 className="text-3xl md:text-3xl font-bold text-slate-800 leading-tight text-center mb-6 relative z-0 drop-shadow-sm">
                {currentQuote.text}
              </h2>
              
              <div className="w-12 h-1 bg-gradient-to-r from-indigo-300 to-purple-300 mx-auto rounded-full mb-6"></div>
              
              <p className="text-center text-lg text-slate-500 font-serif italic">
                — {currentQuote.author}
              </p>
            </div>

            {/* Ações Principais */}
            <div className="mt-10 flex items-center justify-center gap-5 w-full">
              <button 
                onClick={() => toggleFavorite(currentQuote)}
                className={`w-14 h-14 rounded-full flex items-center justify-center shadow-lg transition-all transform hover:scale-105 active:scale-95 ${
                  favorites.some(f => f.id === currentQuote.id)
                    ? 'bg-red-500 text-white shadow-red-200 ring-2 ring-red-100'
                    : 'bg-white text-slate-400 hover:text-red-500 shadow-slate-200 border border-slate-50'
                }`}
                title="Favoritar"
              >
                <Heart size={24} fill={favorites.some(f => f.id === currentQuote.id) ? "currentColor" : "none"} />
              </button>

              {currentView === 'explore' ? (
                <button 
                  onClick={setRandomQuote}
                  className="bg-indigo-600 text-white px-8 h-14 rounded-full shadow-lg shadow-indigo-200 font-bold text-lg flex items-center gap-2 hover:bg-indigo-700 transition-all transform hover:scale-105 active:scale-95 ring-2 ring-indigo-100"
                >
                  <RefreshCw size={20} />
                  Nova Frase
                </button>
              ) : (
                <button 
                  onClick={() => {setRandomQuote();}}
                  className="bg-white text-indigo-600 border border-indigo-100 px-6 h-14 rounded-full shadow-lg shadow-slate-100 font-bold flex items-center gap-2 hover:bg-indigo-50 transition-all transform hover:scale-105"
                >
                  <Sparkles size={20} />
                  Explorar
                </button>
              )}

              <button 
                onClick={() => copyToClipboard(currentQuote.text, currentQuote.author)}
                className="w-14 h-14 bg-white text-slate-400 rounded-full shadow-lg shadow-slate-200 hover:text-indigo-600 flex items-center justify-center transition-all transform hover:scale-105 active:scale-95 border border-slate-50"
                title="Copiar"
              >
                <Share2 size={24} />
              </button>
            </div>

            {/* --- ÁREA DE ANÚNCIO (Banner Mobile) --- */}
            <div className="mt-auto w-full pt-8">
               <AdBanner type="standard" />
            </div>

          </div>
        )}
      </main>

      {/* Navegação Tab Bar */}
      <nav className="bg-white border-t border-slate-100 flex justify-around items-center px-2 pb-safe pt-1 absolute bottom-0 w-full z-20 shadow-[0_-5px_20px_rgba(0,0,0,0.03)]">
        <NavButton view="daily" icon={Calendar} label="Hoje" onClick={setDailyQuote} />
        <NavButton view="explore" icon={RefreshCw} label="Explorar" onClick={setRandomQuote} />
        <NavButton view="favorites" icon={Heart} label="Favoritos" onClick={() => setCurrentView('favorites')} />
      </nav>

      {/* Estilos CSS */}
      <style>{`
        @keyframes fade-in {
          from { opacity: 0; transform: translateY(10px); }
          to { opacity: 1; transform: translateY(0); }
        }
        @keyframes fade-in-down {
          from { opacity: 0; transform: translate(-50%, -20px); }
          to { opacity: 1; transform: translate(-50%, 0); }
        }
        .animate-fade-in {
          animation: fade-in 0.4s ease-out forwards;
        }
        .animate-fade-in-down {
          animation: fade-in-down 0.3s ease-out forwards;
        }
        .pb-safe {
          padding-bottom: max(env(safe-area-inset-bottom), 16px);
        }
      `}</style>
    </div>
  );
}
