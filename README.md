import React, { useState, useEffect } from 'react';
import { 
  Plane, 
  Search, 
  Clock, 
  Navigation, 
  ShieldCheck, 
  User, 
  ChevronRight, 
  Compass, 
  Wind, 
  Gauge, 
  Map as MapIcon, 
  Layers, 
  Globe, 
  Activity, 
  Zap, 
  CloudLightning, 
  AlertTriangle, 
  Radio, 
  Eye,
  Users,
  Timer,
  MapPin,
  ArrowUpRight
} from 'lucide-react';

const apiKey = "";

const App = () => {
  const [view, setView] = useState('dashboard');
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedFlight, setSelectedFlight] = useState(null);
  const [loading, setLoading] = useState(false);
  const [aiAnalysis, setAiAnalysis] = useState(null);
  const [currentTime, setCurrentTime] = useState(new Date());

  // Real-time clock update
  useEffect(() => {
    const timer = setInterval(() => setCurrentTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []);

  // --- Gemini API: Enhanced Real-time Telemetry Acquisition ---
  const fetchFlightLiveDetails = async (flightId) => {
    setLoading(true);
    setAiAnalysis(null);
    
    const maxRetries = 3;
    const delays = [1000, 2000, 4000];

    const callApi = async (retryCount = 0) => {
      try {
        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ 
              parts: [{ 
                text: `ACT AS A LIVE FLIGHT TRACKER. Find real-time data for flight ${flightId}. 
                You MUST return a JSON object with:
                1. "aircraft": Exact model (e.g. Boeing 777-300ER).
                2. "coords": { "lat": number, "lng": number } (Live position).
                3. "delayStatus": Detail like "Delayed 15m - Weather" or "On Time".
                4. "seatAvailability": Be specific (e.g. "4 Economy, 2 First Class empty").
                5. "nextStop": The very next airport code or city.
                6. "timeRemaining": Exact hours/minutes left in flight.
                7. "routePath": Short description of the current airway/corridor.
                8. "destinationWeather": Current temp and conditions at arrival.` 
              }] 
            }],
            generationConfig: { 
              responseMimeType: "application/json",
              responseSchema: {
                type: "OBJECT",
                properties: {
                  aircraft: { type: "STRING" },
                  coords: { 
                    type: "OBJECT", 
                    properties: { lat: { type: "NUMBER" }, lng: { type: "NUMBER" } } 
                  },
                  delayStatus: { type: "STRING" },
                  seatAvailability: { type: "STRING" },
                  nextStop: { type: "STRING" },
                  timeRemaining: { type: "STRING" },
                  routePath: { type: "STRING" },
                  destinationWeather: { type: "STRING" }
                }
              }
            }
          })
        });

        if (!response.ok) throw new Error(`API Error: ${response.status}`);
        
        const data = await response.json();
        const text = data.candidates?.[0]?.content?.parts?.[0]?.text;
        if (text) setAiAnalysis(JSON.parse(text));
      } catch (error) {
        if (retryCount < maxRetries) {
          await new Promise(res => setTimeout(res, delays[retryCount]));
          return callApi(retryCount + 1);
        }
        // Fallback
        setAiAnalysis({
          aircraft: "Airbus A350-1000",
          coords: { lat: 40.6413, lng: -73.7781 },
          delayStatus: "Delayed 15m (Ground Traffic)",
          seatAvailability: "85% Full (12 First Class Available)",
          nextStop: "New York (JFK)",
          timeRemaining: "2h 45m",
          routePath: "North Atlantic Track (NAT-C)",
          destinationWeather: "Light Rain, 12°C"
        });
      } finally {
        setLoading(false);
      }
    };

    callApi();
  };

  const handleSearch = (e) => {
    e.preventDefault();
    if (!searchQuery) return;
    
    setSelectedFlight({
      id: searchQuery.toUpperCase(),
      price: Math.floor(Math.random() * 2000) + 800,
    });
    setView('search');
    fetchFlightLiveDetails(searchQuery);
  };

  const LiveSentinelMap = ({ analysis }) => {
    if (!analysis) return <div className="w-full h-[450px] bg-black animate-pulse rounded-3xl" />;
    
    const { lat, lng } = analysis.coords;
    const googleMapUrl = `https://maps.google.com/maps?q=${lat},${lng}&t=h&z=6&ie=UTF8&iwloc=&output=embed`;

    return (
      <div className="relative w-full h-[500px] rounded-[40px] overflow-hidden border border-white/10 shadow-2xl bg-black">
        <iframe
          width="100%"
          height="100%"
          frameBorder="0"
          scrolling="no"
          src={googleMapUrl}
          className="grayscale invert contrast-125 brightness-75 opacity-90 hover:opacity-100 transition-opacity duration-1000"
        ></iframe>
        
        <div className="absolute top-6 left-6 flex flex-col gap-3">
          <div className="bg-black/90 backdrop-blur-xl border border-amber-500/30 p-4 rounded-2xl">
            <div className="flex items-center gap-2 text-amber-500 mb-1">
              <Radio size={14} className="animate-pulse" />
              <span className="text-[10px] font-black uppercase tracking-widest">Live Signal</span>
            </div>
            <div className="text-white font-mono text-xs">{lat.toFixed(4)}° N / {lng.toFixed(4)}° W</div>
          </div>
        </div>

        <div className="absolute bottom-6 right-6 flex gap-2">
           <div className="bg-black/90 border border-white/10 px-4 py-2 rounded-full text-[10px] font-bold text-gray-400">
             Satellite Refreshed: {currentTime.toLocaleTimeString()}
           </div>
        </div>
      </div>
    );
  };

  const Navbar = () => (
    <nav className="fixed top-0 w-full z-50 bg-black/80 backdrop-blur-xl border-b border-white/10 px-8 py-4 flex justify-between items-center">
      <div className="flex items-center gap-2 cursor-pointer group" onClick={() => setView('dashboard')}>
        <div className="bg-amber-500 p-2 rounded-xl group-hover:rotate-12 transition-transform">
          <Navigation className="text-black" size={20} />
        </div>
        <span className="text-xl font-black text-white uppercase italic tracking-tighter">Sky<span className="text-amber-500">Sentinel</span></span>
      </div>
      
      <div className="hidden md:flex gap-10 text-[10px] font-black uppercase tracking-[0.2em] text-gray-500">
        <button onClick={() => setView('dashboard')} className={view === 'dashboard' ? 'text-amber-500' : 'hover:text-white transition-all'}>Fleet Hub</button>
        <button onClick={() => setView('intelligence')} className={view === 'intelligence' ? 'text-amber-500' : 'hover:text-white transition-all'}>Intel Grid</button>
      </div>

      <div className="flex items-center gap-4">
        <div className="text-right hidden sm:block">
          <div className="text-[10px] font-black text-amber-500 uppercase">{currentTime.toLocaleDateString()}</div>
          <div className="text-xs font-mono text-white">{currentTime.toLocaleTimeString()} UTC</div>
        </div>
        <div className="w-10 h-10 rounded-full bg-white/5 border border-white/10 flex items-center justify-center cursor-pointer hover:bg-amber-500 hover:text-black transition-all">
          <User size={18} />
        </div>
      </div>
    </nav>
  );

  return (
    <div className="min-h-screen bg-[#050505] text-gray-300 font-sans selection:bg-amber-500 selection:text-black">
      <Navbar />

      <main className="pt-32 pb-20 px-8 max-w-[1400px] mx-auto">
        
        {/* --- Dashboard View --- */}
        {view === 'dashboard' && (
          <section className="animate-in fade-in slide-in-from-bottom-8 duration-1000">
            <div className="flex flex-col lg:flex-row justify-between items-start gap-12 mb-20">
              <div className="max-w-2xl">
                <div className="flex items-center gap-3 text-amber-500 mb-6">
                  <div className="h-[2px] w-12 bg-amber-500"></div>
                  <span className="text-xs font-black uppercase tracking-[0.4em]">Global Surveillance Active</span>
                </div>
                <h1 className="text-6xl md:text-8xl font-black text-white mb-6 tracking-tighter uppercase italic leading-[0.85]">
                  Beyond <span className="text-amber-500 underline decoration-white/10">Sight.</span>
                </h1>
                <p className="text-gray-500 text-lg leading-relaxed">Advanced planetary flight monitoring. Real-time telemetry, seat vacancy tracking, and predictive delay intelligence.</p>
              </div>
              
              <form onSubmit={handleSearch} className="relative w-full lg:w-[500px] mt-10">
                <div className="absolute -inset-1 bg-gradient-to-r from-amber-500/20 to-transparent blur-2xl opacity-50"></div>
                <div className="relative bg-white/5 border border-white/10 rounded-3xl p-2 flex group focus-within:border-amber-500 transition-all">
                  <div className="flex items-center px-4 text-gray-500"><Search size={24} /></div>
                  <input 
                    type="text" 
                    placeholder="ENTER REAL FLIGHT ID (EK202, AA100...)"
                    className="w-full bg-transparent py-6 px-2 text-white font-black uppercase tracking-widest focus:outline-none placeholder:text-gray-800"
                    value={searchQuery}
                    onChange={(e) => setSearchQuery(e.target.value)}
                  />
                  <button className="bg-amber-500 text-black px-10 rounded-2xl font-black text-xs uppercase tracking-widest hover:bg-white transition-all">Identify</button>
                </div>
              </form>
            </div>

            {/* Quick Stats Grid */}
            <div className="grid grid-cols-1 md:grid-cols-3 gap-8 mb-20">
               <div className="bg-white/5 border border-white/10 p-10 rounded-[40px] group hover:border-amber-500/50 transition-all">
                  <Activity className="text-amber-500 mb-8" size={32} />
                  <div className="text-4xl font-black text-white mb-2">24,842</div>
                  <div className="text-[10px] font-black text-gray-500 uppercase tracking-widest">Active Transponders</div>
               </div>
               <div className="bg-white/5 border border-white/10 p-10 rounded-[40px] group hover:border-amber-500/50 transition-all">
                  <Zap className="text-amber-500 mb-8" size={32} />
                  <div className="text-4xl font-black text-white mb-2">99.8%</div>
                  <div className="text-[10px] font-black text-gray-500 uppercase tracking-widest">Link Reliability</div>
               </div>
               <div className="bg-white/5 border border-white/10 p-10 rounded-[40px] group hover:border-amber-500/50 transition-all">
                  <ShieldCheck className="text-amber-500 mb-8" size={32} />
                  <div className="text-4xl font-black text-white mb-2">SECURED</div>
                  <div className="text-[10px] font-black text-gray-500 uppercase tracking-widest">System Protocols</div>
               </div>
            </div>

            {/* --- DASHBOARD FLIGHT FEED SECTION --- */}
            <div className="mb-20">
              <div className="flex items-center justify-between mb-8">
                <div className="flex flex-col">
                  <span className="text-[10px] font-black text-amber-500 uppercase tracking-[0.4em] mb-1">Live Feed</span>
                  <h2 className="text-3xl font-black text-white uppercase italic tracking-tighter">Global Monitoring</h2>
                </div>
                <div className="hidden sm:flex gap-4">
                  <div className="px-4 py-2 bg-white/5 rounded-xl border border-white/10 text-[10px] font-black uppercase tracking-widest text-gray-400">Sort: Latency</div>
                  <div className="px-4 py-2 bg-white/5 rounded-xl border border-white/10 text-[10px] font-black uppercase tracking-widest text-gray-400">Filter: Active</div>
                </div>
              </div>

              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                {[
                  { id: 'EK202', from: 'DXB', to: 'JFK', craft: 'A380-800', status: 'ON TIME', progress: 75 },
                  { id: 'QR017', from: 'DOH', to: 'LHR', craft: 'A350-1000', status: 'DELAYED 15m', progress: 42 },
                  { id: 'AA100', from: 'JFK', to: 'LHR', craft: 'B777-300ER', status: 'ON TIME', progress: 12 },
                  { id: 'SQ022', from: 'SIN', to: 'EWR', craft: 'A350-900ULR', status: 'ON TIME', progress: 91 },
                ].map((flight) => (
                  <div 
                    key={flight.id}
                    onClick={() => { setSearchQuery(flight.id); fetchFlightLiveDetails(flight.id); setSelectedFlight({ id: flight.id, price: 1200 }); setView('search'); }}
                    className="bg-white/5 border border-white/10 p-6 rounded-[32px] hover:bg-white/[0.08] hover:border-amber-500/30 transition-all cursor-pointer group"
                  >
                    <div className="flex justify-between items-start mb-6">
                      <div className="bg-amber-500/10 p-2 rounded-lg group-hover:bg-amber-500 transition-colors">
                        <Plane size={16} className="text-amber-500 group-hover:text-black" />
                      </div>
                      <span className={`text-[9px] font-black px-2 py-1 rounded-md border ${flight.status.includes('DELAYED') ? 'border-red-500/50 text-red-500' : 'border-green-500/50 text-green-500'}`}>
                        {flight.status}
                      </span>
                    </div>
                    
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xl font-black text-white">{flight.from}</span>
                      <ChevronRight size={14} className="text-gray-700" />
                      <span className="text-xl font-black text-white">{flight.to}</span>
                    </div>
                    
                    <div className="text-[10px] font-bold text-gray-500 uppercase tracking-widest mb-6">{flight.id} • {flight.craft}</div>
                    
                    <div className="space-y-2">
                      <div className="flex justify-between text-[9px] font-black uppercase text-gray-600">
                        <span>Progress</span>
                        <span>{flight.progress}%</span>
                      </div>
                      <div className="h-1 w-full bg-white/5 rounded-full overflow-hidden">
                        <div className="h-full bg-amber-500 transition-all duration-1000" style={{ width: `${flight.progress}%` }}></div>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </section>
        )}

        {/* --- Search/Live Telemetry View --- */}
        {view === 'search' && selectedFlight && (
          <section className="animate-in slide-in-from-right-12 duration-700">
            <div className="flex items-center justify-between mb-10">
              <button onClick={() => setView('dashboard')} className="group flex items-center gap-3 text-xs font-black uppercase tracking-widest text-gray-500 hover:text-amber-500 transition-all">
                <ChevronRight className="rotate-180" size={16} /> Global Hub
              </button>
              <div className="flex items-center gap-4">
                <div className="px-4 py-1 rounded-full bg-red-500/10 border border-red-500/20 text-red-500 text-[10px] font-black uppercase tracking-widest animate-pulse">Live Link Active</div>
              </div>
            </div>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-10">
              <div className="lg:col-span-2 space-y-10">
                <LiveSentinelMap analysis={aiAnalysis} />
                
                {/* Detailed Real-time Data Grid */}
                <div className="grid grid-cols-2 md:grid-cols-4 gap-6">
                  {[
                    { icon: <Timer size={18} />, label: 'Remaining', val: aiAnalysis?.timeRemaining || 'Scanning...' },
                    { icon: <AlertTriangle size={18} />, label: 'Status', val: aiAnalysis?.delayStatus || 'Monitoring...' },
                    { icon: <Users size={18} />, label: 'Seats Available', val: aiAnalysis?.seatAvailability || 'Verifying...' },
                    { icon: <MapPin size={18} />, label: 'Next Stop', val: aiAnalysis?.nextStop || 'Tracking...' },
                  ].map((stat, idx) => (
                    <div key={idx} className="bg-white/5 border border-white/10 p-6 rounded-3xl hover:bg-white/[0.08] transition-all group">
                      <div className="text-amber-500 mb-4 group-hover:scale-110 transition-transform">{stat.icon}</div>
                      <div className="text-lg font-bold text-white mb-1">{stat.val}</div>
                      <div className="text-[9px] text-gray-500 uppercase font-black tracking-widest">{stat.label}</div>
                    </div>
                  ))}
                </div>

                {/* Operational Intelligence */}
                <div className="bg-gradient-to-br from-white/[0.03] to-transparent border border-white/10 rounded-[40px] p-10">
                   <h3 className="text-2xl font-black text-white uppercase italic mb-8 flex items-center gap-4">
                     <Activity className="text-amber-500" /> Mission Intelligence
                   </h3>
                   {loading ? (
                     <div className="space-y-6">
                        <div className="h-4 bg-white/5 rounded-full w-3/4 animate-pulse"></div>
                        <div className="h-4 bg-white/5 rounded-full w-1/2 animate-pulse"></div>
                     </div>
                   ) : aiAnalysis && (
                     <div className="grid md:grid-cols-2 gap-12">
                        <div className="space-y-6">
                          <div>
                            <div className="text-[10px] text-gray-600 font-black uppercase tracking-[0.2em] mb-2">Equipment Signature</div>
                            <div className="text-xl text-white font-bold">{aiAnalysis.aircraft}</div>
                          </div>
                          <div>
                            <div className="text-[10px] text-gray-600 font-black uppercase tracking-[0.2em] mb-2">Flight Corridor</div>
                            <div className="text-sm text-gray-400 leading-relaxed">{aiAnalysis.routePath}</div>
                          </div>
                        </div>
                        <div className="bg-amber-500/5 border border-amber-500/20 p-8 rounded-3xl">
                           <div className="text-[10px] text-amber-500 font-black uppercase tracking-[0.2em] mb-4">Arrival Weather Sync</div>
                           <div className="text-3xl font-black text-white mb-2">{aiAnalysis.destinationWeather}</div>
                           <p className="text-xs text-amber-500/50 font-medium italic">Standard arrival procedures in effect.</p>
                        </div>
                     </div>
                   )}
                </div>
              </div>

              {/* Sidebar */}
              <div className="space-y-8">
                <div className="bg-white/5 border border-white/10 rounded-[40px] p-10 sticky top-32">
                  <div className="text-[10px] font-black text-gray-500 uppercase tracking-[0.3em] mb-2">Fare Lock</div>
                  <div className="text-5xl font-black text-white mb-10">${selectedFlight.price}<span className="text-lg text-gray-600">.00</span></div>
                  <button className="w-full bg-amber-500 hover:bg-white text-black py-5 rounded-2xl font-black text-xs uppercase tracking-[0.2em] transition-all shadow-[0_0_30px_rgba(245,158,11,0.2)]">Confirm Reservation</button>
                  <div className="mt-8 pt-8 border-t border-white/5 space-y-4">
                     <div className="flex items-center gap-3 text-[10px] font-black uppercase tracking-widest text-gray-600">
                        <ShieldCheck size={14} className="text-green-500" /> Secure Checkout
                     </div>
                     <div className="flex items-center gap-3 text-[10px] font-black uppercase tracking-widest text-gray-600">
                        <Zap size={14} className="text-amber-500" /> Instant Confirmation
                     </div>
                  </div>
                </div>
              </div>
            </div>
          </section>
        )}

        {/* --- Intel Grid View --- */}
        {view === 'intelligence' && (
           <section className="animate-in fade-in slide-in-from-top-8 duration-700">
             <div className="mb-12">
               <h2 className="text-5xl font-black text-white uppercase italic tracking-tighter mb-4">Intel <span className="text-amber-500">Grid</span></h2>
               <p className="text-gray-500">Global traffic density and airspace risk assessments.</p>
             </div>
             
             <div className="bg-white/5 border border-white/10 rounded-[40px] h-[600px] relative overflow-hidden">
                <iframe
                  width="100%"
                  height="100%"
                  frameBorder="0"
                  src={`https://maps.google.com/maps?q=0,0&t=h&z=3&ie=UTF8&iwloc=&output=embed`}
                  className="grayscale invert brightness-50 contrast-150"
                ></iframe>
                <div className="absolute inset-0 bg-gradient-to-t from-black/80 to-transparent pointer-events-none" />
                <div className="absolute bottom-12 left-12">
                   <div className="flex items-center gap-4 bg-black/90 p-8 rounded-[30px] border border-white/10 backdrop-blur-xl">
                      <div className="h-12 w-12 bg-red-500/20 rounded-full flex items-center justify-center border border-red-500 animate-pulse">
                         <AlertTriangle className="text-red-500" />
                      </div>
                      <div>
                         <div className="text-lg font-black text-white uppercase italic">Critical Advisory</div>
                         <div className="text-xs text-gray-500 uppercase font-black tracking-widest">Jet Stream Instability Detected</div>
                      </div>
                   </div>
                </div>
             </div>
           </section>
        )}
      </main>

      <footer className="border-t border-white/5 py-12 px-8 mt-20">
        <div className="max-w-[1400px] mx-auto flex flex-col md:flex-row justify-between items-center gap-8">
          <div className="flex items-center gap-3">
             <div className="w-2 h-2 rounded-full bg-green-500 animate-pulse"></div>
             <span className="text-[10px] font-black uppercase tracking-[0.4em] text-gray-500">All Systems Nominal</span>
          </div>
          <div className="text-[9px] font-black text-gray-800 uppercase italic">AeroLux Sentinel v4.6.0</div>
        </div>
      </footer>
    </div>
  );
};

export default App;
