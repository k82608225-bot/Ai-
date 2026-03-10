import { useState, useEffect, useRef } from "react";

const CATEGORIES = ["hashtags", "audio", "anime"];

const SYSTEM_PROMPT = `You are a viral content trend expert specializing in short-form video editing (Reels, TikTok, YouTube Shorts). 
When asked, return ONLY a valid JSON object (no markdown, no explanation) in this exact format:
{
  "hashtags": [
    {"tag": "#hashtag", "heat": 95, "niche": "edits"},
    ...10 items
  ],
  "audio": [
    {"name": "Song Name - Artist", "heat": 92, "genre": "phonk", "vibe": "dark cinematic"},
    ...10 items
  ],
  "anime": [
    {"name": "Character Name", "anime": "Anime Title", "heat": 88, "editStyle": "AMV/baddie/sigma"},
    ...10 items
  ]
}

Focus on what's actually trending RIGHT NOW for video editors on TikTok/Reels/YouTube Shorts. Include viral edit hashtags, trending phonk/dark/anime audio, and popular anime characters for edits.`;

export default function TrendBot() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [activeTab, setActiveTab] = useState("hashtags");
  const [error, setError] = useState(null);
  const [copied, setCopied] = useState(null);
  const [query, setQuery] = useState("");
  const particlesRef = useRef([]);

  const fetchTrends = async (customQuery = "") => {
    setLoading(true);
    setError(null);
    setData(null);
    try {
      const userMsg = customQuery
        ? `Give me trending ${customQuery} for viral video edits right now`
        : "Give me all the most trending hashtags, audio tracks, and anime characters for viral video edits right now in 2025";

      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: SYSTEM_PROMPT,
          messages: [{ role: "user", content: userMsg }],
        }),
      });
      const json = await res.json();
      const text = json.content?.map(b => b.text || "").join("") || "";
      const clean = text.replace(/```json|```/g, "").trim();
      const parsed = JSON.parse(clean);
      setData(parsed);
    } catch (e) {
      setError("Failed to fetch trends. Try again!");
    }
    setLoading(false);
  };

  useEffect(() => { fetchTrends(); }, []);

  const copyToClipboard = (text, id) => {
    navigator.clipboard.writeText(text);
    setCopied(id);
    setTimeout(() => setCopied(null), 1500);
  };

  const heatColor = (h) => {
    if (h >= 90) return "#ff3cac";
    if (h >= 75) return "#ff8c00";
    return "#00f5a0";
  };

  const heatLabel = (h) => {
    if (h >= 90) return "🔥 VIRAL";
    if (h >= 75) return "⚡ HOT";
    return "📈 RISING";
  };

  return (
    <div style={{
      minHeight: "100vh",
      background: "#030007",
      fontFamily: "'Orbitron', 'Rajdhani', monospace",
      color: "#fff",
      padding: "20px",
      position: "relative",
      overflow: "hidden",
    }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Rajdhani:wght@400;600;700&display=swap');

        * { box-sizing: border-box; }

        .bg-grid {
          position: fixed; top: 0; left: 0; right: 0; bottom: 0;
          background-image: 
            linear-gradient(rgba(255,60,172,0.05) 1px, transparent 1px),
            linear-gradient(90deg, rgba(255,60,172,0.05) 1px, transparent 1px);
          background-size: 40px 40px;
          pointer-events: none; z-index: 0;
        }

        .glow-orb {
          position: fixed; border-radius: 50%;
          pointer-events: none; z-index: 0; filter: blur(80px);
        }

        .card {
          background: rgba(255,255,255,0.03);
          border: 1px solid rgba(255,60,172,0.2);
          border-radius: 16px;
          padding: 18px 20px;
          margin-bottom: 12px;
          display: flex; align-items: center; gap: 16px;
          position: relative; overflow: hidden;
          transition: all 0.3s ease;
          cursor: pointer;
          animation: fadeSlide 0.4s ease forwards;
          opacity: 0;
        }
        .card:hover {
          border-color: rgba(255,60,172,0.6);
          background: rgba(255,60,172,0.06);
          transform: translateX(4px);
        }
        .card::before {
          content: '';
          position: absolute; left: 0; top: 0; bottom: 0; width: 3px;
          background: linear-gradient(to bottom, #ff3cac, #784ba0);
        }

        @keyframes fadeSlide {
          from { opacity: 0; transform: translateX(-20px); }
          to { opacity: 1; transform: translateX(0); }
        }

        .tab-btn {
          padding: 10px 24px; border: none; cursor: pointer;
          font-family: 'Orbitron', monospace; font-size: 11px;
          font-weight: 700; letter-spacing: 2px; text-transform: uppercase;
          border-radius: 8px; transition: all 0.2s ease;
        }

        .heat-bar {
          height: 4px; border-radius: 2px;
          background: rgba(255,255,255,0.1);
          overflow: hidden; margin-top: 6px;
        }
        .heat-fill {
          height: 100%; border-radius: 2px;
          transition: width 1s ease;
        }

        .copy-btn {
          margin-left: auto; padding: 6px 14px;
          background: rgba(255,60,172,0.15);
          border: 1px solid rgba(255,60,172,0.4);
          border-radius: 6px; color: #ff3cac;
          font-size: 11px; font-family: 'Rajdhani', sans-serif;
          font-weight: 700; letter-spacing: 1px;
          cursor: pointer; white-space: nowrap;
          transition: all 0.2s;
        }
        .copy-btn:hover { background: rgba(255,60,172,0.3); }

        .fetch-btn {
          background: linear-gradient(135deg, #ff3cac, #784ba0, #2b86c5);
          border: none; border-radius: 12px; color: white;
          font-family: 'Orbitron', monospace; font-weight: 700;
          font-size: 12px; letter-spacing: 2px; cursor: pointer;
          padding: 14px 32px; transition: all 0.3s ease;
          position: relative; overflow: hidden;
        }
        .fetch-btn:hover { transform: scale(1.04); box-shadow: 0 0 30px rgba(255,60,172,0.5); }
        .fetch-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }

        .pulse-ring {
          display: inline-block; width: 12px; height: 12px;
          border-radius: 50%; background: #ff3cac;
          animation: pulse 1.2s ease infinite;
        }
        @keyframes pulse {
          0%, 100% { transform: scale(1); opacity: 1; }
          50% { transform: scale(1.5); opacity: 0.5; }
        }

        .loading-dots span {
          display: inline-block; width: 8px; height: 8px;
          border-radius: 50%; background: #ff3cac; margin: 0 3px;
          animation: bounce 1.2s ease infinite;
        }
        .loading-dots span:nth-child(2) { animation-delay: 0.2s; background: #784ba0; }
        .loading-dots span:nth-child(3) { animation-delay: 0.4s; background: #2b86c5; }
        @keyframes bounce {
          0%, 100% { transform: translateY(0); }
          50% { transform: translateY(-10px); }
        }

        .rank-badge {
          width: 32px; height: 32px; border-radius: 8px;
          display: flex; align-items: center; justify-content: center;
          font-weight: 900; font-size: 13px; flex-shrink: 0;
        }

        .search-input {
          background: rgba(255,255,255,0.05);
          border: 1px solid rgba(255,60,172,0.3);
          border-radius: 10px; color: white; padding: 12px 18px;
          font-family: 'Rajdhani', sans-serif; font-size: 15px;
          outline: none; width: 100%; transition: border-color 0.2s;
        }
        .search-input:focus { border-color: rgba(255,60,172,0.8); }
        .search-input::placeholder { color: rgba(255,255,255,0.3); }
      `}</style>

      <div className="bg-grid" />
      <div className="glow-orb" style={{ width: 400, height: 400, top: -100, right: -100, background: "rgba(255,60,172,0.15)" }} />
      <div className="glow-orb" style={{ width: 300, height: 300, bottom: 0, left: -100, background: "rgba(43,134,197,0.1)" }} />

      <div style={{ maxWidth: 720, margin: "0 auto", position: "relative", zIndex: 1 }}>
        {/* Header */}
        <div style={{ textAlign: "center", marginBottom: 36 }}>
          <div style={{ display: "flex", alignItems: "center", justifyContent: "center", gap: 12, marginBottom: 8 }}>
            <div className="pulse-ring" />
            <span style={{ fontSize: 11, letterSpacing: 4, color: "#ff3cac", fontWeight: 700 }}>LIVE TREND SCANNER</span>
            <div className="pulse-ring" />
          </div>
          <h1 style={{
            fontSize: "clamp(28px, 6vw, 52px)", fontWeight: 900, margin: 0,
            background: "linear-gradient(135deg, #ff3cac 0%, #784ba0 50%, #2b86c5 100%)",
            WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent",
            letterSpacing: 2, lineHeight: 1.1,
          }}>
            VIRAL EDIT BOT
          </h1>
          <p style={{ color: "rgba(255,255,255,0.4)", marginTop: 10, fontSize: 14, fontFamily: "'Rajdhani', sans-serif", letterSpacing: 1 }}>
            AI-powered trending hashtags • audio • anime for your edits
          </p>
        </div>

        {/* Search + Fetch */}
        <div style={{ display: "flex", gap: 12, marginBottom: 28 }}>
          <input
            className="search-input"
            placeholder="Search specific trends... (e.g. 'dark phonk audio' or 'shonen anime')"
            value={query}
            onChange={e => setQuery(e.target.value)}
            onKeyDown={e => e.key === "Enter" && fetchTrends(query)}
          />
          <button className="fetch-btn" onClick={() => fetchTrends(query)} disabled={loading}>
            {loading ? "..." : "SCAN"}
          </button>
        </div>

        {/* Loading */}
        {loading && (
          <div style={{ textAlign: "center", padding: "60px 0" }}>
            <div className="loading-dots" style={{ marginBottom: 20 }}>
              <span /><span /><span />
            </div>
            <div style={{ color: "rgba(255,255,255,0.4)", fontSize: 13, letterSpacing: 2 }}>SCANNING VIRAL TRENDS...</div>
          </div>
        )}

        {/* Error */}
        {error && (
          <div style={{ textAlign: "center", padding: 40, color: "#ff6b6b" }}>
            <div style={{ fontSize: 32, marginBottom: 12 }}>⚠️</div>
            <div>{error}</div>
            <button className="fetch-btn" onClick={() => fetchTrends()} style={{ marginTop: 20 }}>RETRY</button>
          </div>
        )}

        {/* Tabs + Data */}
        {data && !loading && (
          <>
            {/* Tabs */}
            <div style={{ display: "flex", gap: 8, marginBottom: 24, overflowX: "auto", paddingBottom: 4 }}>
              {[
                { key: "hashtags", icon: "#", label: "HASHTAGS" },
                { key: "audio", icon: "♪", label: "AUDIO" },
                { key: "anime", icon: "⚔", label: "ANIME" },
              ].map(t => (
                <button
                  key={t.key}
                  className="tab-btn"
                  onClick={() => setActiveTab(t.key)}
                  style={{
                    background: activeTab === t.key
                      ? "linear-gradient(135deg, #ff3cac, #784ba0)"
                      : "rgba(255,255,255,0.05)",
                    color: activeTab === t.key ? "#fff" : "rgba(255,255,255,0.4)",
                    border: activeTab === t.key ? "none" : "1px solid rgba(255,255,255,0.1)",
                    boxShadow: activeTab === t.key ? "0 0 20px rgba(255,60,172,0.4)" : "none",
                  }}
                >
                  {t.icon} {t.label}
                </button>
              ))}

              <button className="fetch-btn" onClick={() => fetchTrends(query)} style={{ marginLeft: "auto", padding: "10px 20px", fontSize: 10 }}>
                🔄 REFRESH
              </button>
            </div>

            {/* Hashtags */}
            {activeTab === "hashtags" && (
              <div>
                <div style={{ marginBottom: 16, color: "rgba(255,255,255,0.3)", fontSize: 12, letterSpacing: 2 }}>
                  TOP {data.hashtags?.length || 0} TRENDING HASHTAGS
                </div>
                {data.hashtags?.map((item, i) => (
                  <div key={i} className="card" style={{ animationDelay: `${i * 0.06}s` }}
                    onClick={() => copyToClipboard(item.tag, `h${i}`)}>
                    <div className="rank-badge" style={{
                      background: i < 3 ? "linear-gradient(135deg, #ff3cac, #784ba0)" : "rgba(255,255,255,0.06)",
                      color: i < 3 ? "#fff" : "rgba(255,255,255,0.4)",
                    }}>
                      {i < 3 ? ["👑","🥈","🥉"][i] : `#${i+1}`}
                    </div>
                    <div style={{ flex: 1, minWidth: 0 }}>
                      <div style={{ fontWeight: 700, fontSize: 17, color: "#fff", marginBottom: 4 }}>{item.tag}</div>
                      <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
                        <span style={{ fontSize: 11, color: heatColor(item.heat), fontWeight: 700, letterSpacing: 1 }}>
                          {heatLabel(item.heat)}
                        </span>
                        <span style={{ fontSize: 11, color: "rgba(255,255,255,0.3)" }}>{item.niche}</span>
                      </div>
                      <div className="heat-bar">
                        <div className="heat-fill" style={{ width: `${item.heat}%`, background: `linear-gradient(90deg, ${heatColor(item.heat)}, #784ba0)` }} />
                      </div>
                    </div>
                    <button className="copy-btn">{copied === `h${i}` ? "✓ COPIED" : "COPY"}</button>
                  </div>
                ))}
                {/* Copy All */}
                <button className="fetch-btn" style={{ width: "100%", marginTop: 8 }}
                  onClick={() => copyToClipboard(data.hashtags.map(h => h.tag).join(" "), "all-hash")}>
                  {copied === "all-hash" ? "✓ ALL COPIED!" : "📋 COPY ALL HASHTAGS"}
                </button>
              </div>
            )}

            {/* Audio */}
            {activeTab === "audio" && (
              <div>
                <div style={{ marginBottom: 16, color: "rgba(255,255,255,0.3)", fontSize: 12, letterSpacing: 2 }}>
                  TOP {data.audio?.length || 0} TRENDING AUDIO TRACKS
                </div>
                {data.audio?.map((item, i) => (
                  <div key={i} className="card" style={{ animationDelay: `${i * 0.06}s` }}
                    onClick={() => copyToClipboard(item.name, `a${i}`)}>
                    <div className="rank-badge" style={{
                      background: i < 3 ? "linear-gradient(135deg, #ff8c00, #ff3cac)" : "rgba(255,255,255,0.06)",
                      color: i < 3 ? "#fff" : "rgba(255,255,255,0.4)",
                    }}>
                      {i < 3 ? ["🎵","🎶","🎸"][i] : `#${i+1}`}
                    </div>
                    <div style={{ flex: 1, minWidth: 0 }}>
                      <div style={{ fontWeight: 700, fontSize: 15, color: "#fff", marginBottom: 4, whiteSpace: "nowrap", overflow: "hidden", textOverflow: "ellipsis" }}>
                        {item.name}
                      </div>
                      <div style={{ display: "flex", gap: 10, flexWrap: "wrap" }}>
                        <span style={{ fontSize: 11, padding: "2px 8px", background: "rgba(255,140,0,0.15)", color: "#ff8c00", borderRadius: 4, fontFamily: "'Rajdhani', sans-serif" }}>
                          {item.genre}
                        </span>
                        <span style={{ fontSize: 11, color: "rgba(255,255,255,0.4)", fontFamily: "'Rajdhani', sans-serif" }}>
                          {item.vibe}
                        </span>
                        <span style={{ fontSize: 11, color: heatColor(item.heat), fontWeight: 700, marginLeft: "auto" }}>
                          {heatLabel(item.heat)}
                        </span>
                      </div>
                      <div className="heat-bar" style={{ marginTop: 8 }}>
                        <div className="heat-fill" style={{ width: `${item.heat}%`, background: "linear-gradient(90deg, #ff8c00, #ff3cac)" }} />
                      </div>
                    </div>
                    <button className="copy-btn">{copied === `a${i}` ? "✓ COPIED" : "COPY"}</button>
                  </div>
                ))}
              </div>
            )}

            {/* Anime */}
            {activeTab === "anime" && (
              <div>
                <div style={{ marginBottom: 16, color: "rgba(255,255,255,0.3)", fontSize: 12, letterSpacing: 2 }}>
                  TOP {data.anime?.length || 0} VIRAL ANIME CHARACTERS
                </div>
                {data.anime?.map((item, i) => (
                  <div key={i} className="card" style={{ animationDelay: `${i * 0.06}s` }}
                    onClick={() => copyToClipboard(`${item.name} - ${item.anime}`, `n${i}`)}>
                    <div className="rank-badge" style={{
                      background: i < 3 ? "linear-gradient(135deg, #2b86c5, #784ba0)" : "rgba(255,255,255,0.06)",
                      color: i < 3 ? "#fff" : "rgba(255,255,255,0.4)",
                    }}>
                      {i < 3 ? ["⚡","🌸","🗡️"][i] : `#${i+1}`}
                    </div>
                    <div style={{ flex: 1, minWidth: 0 }}>
                      <div style={{ fontWeight: 700, fontSize: 17, color: "#fff", marginBottom: 2 }}>{item.name}</div>
                      <div style={{ fontSize: 12, color: "#2b86c5", fontFamily: "'Rajdhani', sans-serif", marginBottom: 4 }}>{item.anime}</div>
                      <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
                        <span style={{ fontSize: 11, padding: "2px 8px", background: "rgba(43,134,197,0.15)", color: "#2b86c5", borderRadius: 4, fontFamily: "'Rajdhani', sans-serif" }}>
                          {item.editStyle}
                        </span>
                        <span style={{ fontSize: 11, color: heatColor(item.heat), fontWeight: 700, marginLeft: "auto" }}>
                          {heatLabel(item.heat)}
                        </span>
                      </div>
                      <div className="heat-bar" style={{ marginTop: 8 }}>
                        <div className="heat-fill" style={{ width: `${item.heat}%`, background: "linear-gradient(90deg, #2b86c5, #784ba0)" }} />
                      </div>
                    </div>
                    <button className="copy-btn">{copied === `n${i}` ? "✓ COPIED" : "COPY"}</button>
                  </div>
                ))}
              </div>
            )}
          </>
        )}

        <div style={{ textAlign: "center", marginTop: 40, color: "rgba(255,255,255,0.15)", fontSize: 11, letterSpacing: 2 }}>
          POWERED BY CLAUDE AI • CLICK ANY CARD TO COPY
        </div>
      </div>
    </div>
  );
}
