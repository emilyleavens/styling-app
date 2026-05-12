import { useState, useEffect } from "react";

const PROFILE = {
  height: "5'6.5\"", waist: "29.5 in", hips: "38 in", inseam: "29 in", chest: "40 in",
  bodyShape: "Inverted triangle / hourglass hybrid (bust 40in, hips 38in, waist 29.5in — defined waist, bust is widest point with strong hip curve)",
  coloring: "Soft Autumn — fair skin with olive (yellow-green) undertones, brown hair, blue eyes",
  colorNotes: "Warm muted earthy tones: camel, warm taupe, dusty terracotta, sage, moss green, chocolate brown, dusty rose, warm ivory, burnt sienna. Blue eyes enhanced by warm rust, terracotta, chocolate. Avoid cool grays, icy pastels, stark white and black.",
  styleProfile: "Romantic minimalist with boho undercurrent. Favorite brands: Reformation (fitted, vintage-inspired, body-conscious), Anthropologie (texture, prints, earthy), Everlane (clean minimalist basics), Abercrombie (elevated casual, barrel jeans), Free People (flowy, layered, free-spirited). She loves collars (Peter Pan, pointed, detachable, button-downs) and vests (sweater vests, tailored, longline knit, puffer). She loves boxy cropped tops — always pair with high-waisted bottoms so the bottom does the waist-definition work.",
  fitNotes: "Hourglass/inverted triangle: always define the waist with high-waisted bottoms when top is boxy/loose, no boxy full-length tops, wide-leg or flared bottoms, wrap styles, V-necks. Shorter inseam (29in): ankle or cropped hems preferred."
};

const TRENDS = `2026 trends: quiet luxury/elevated minimalism, barrel-leg and wide-leg trousers, relaxed blazers, ballet flats/loafers/mary janes/kitten heels, maxi/midi skirts, soft drapey fabrics (satin, silk-look, soft knit), lightweight knit layering, longline cardigans, trench and tailored coats. Denim: wide-leg, barrel, straight — NOT skinny.`;

async function callClaude(messages, maxTokens = 1000, tools = []) {
  const body = { model: "claude-sonnet-4-20250514", max_tokens: maxTokens, messages };
  if (tools.length) body.tools = tools;
  const res = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(body)
  });
  if (!res.ok) throw new Error("API error");
  const data = await res.json();
  return (data.content || []).filter(b => b.type === "text").map(b => b.text).join("\n");
}

function extractJSON(text) {
  const match = text.match(/\{[\s\S]*\}/);
  if (!match) throw new Error("No JSON");
  return JSON.parse(match[0]);
}

function weatherIcon(code) {
  if (code === 0) return "ti-sun";
  if (code <= 2) return "ti-cloud-sun";
  if (code <= 3) return "ti-cloud";
  if (code <= 67) return "ti-cloud-rain";
  if (code <= 77) return "ti-snowflake";
  if (code <= 82) return "ti-cloud-storm";
  return "ti-storm";
}

function weatherDesc(code) {
  if (code === 0) return "Clear";
  if (code === 1) return "Mostly clear";
  if (code === 2) return "Partly cloudy";
  if (code === 3) return "Overcast";
  if (code <= 49) return "Foggy";
  if (code <= 55) return "Drizzle";
  if (code <= 67) return "Rain";
  if (code <= 77) return "Snow";
  if (code <= 82) return "Showers";
  return "Stormy";
}

const Tag = ({ children, accent }) => (
  <span style={{
    fontSize: 11, fontWeight: 500, letterSpacing: "0.06em", textTransform: "uppercase",
    padding: "3px 10px", borderRadius: 20,
    background: accent ? "var(--color-text-primary)" : "var(--color-background-tertiary)",
    color: accent ? "var(--color-background-primary)" : "var(--color-text-tertiary)",
    border: accent ? "none" : "0.5px solid var(--color-border-tertiary)"
  }}>{children}</span>
);

const Divider = () => <div style={{ height: "0.5px", background: "var(--color-border-tertiary)", margin: "20px 0" }} />;

const Label = ({ children }) => (
  <div style={{ fontSize: 11, fontWeight: 500, letterSpacing: "0.08em", textTransform: "uppercase", color: "var(--color-text-tertiary)", marginBottom: 10 }}>{children}</div>
);

export default function App() {
  const [step, setStep] = useState("form");
  const [activity, setActivity] = useState("");
  const [notes, setNotes] = useState("");
  const [result, setResult] = useState(null);
  const [error, setError] = useState(null);
  const [loadingMsg, setLoadingMsg] = useState("");

  const [locationInput, setLocationInput] = useState("Denver, Colorado");
  const [locationLabel, setLocationLabel] = useState("Denver, Colorado, US");
  const [locationCoords, setLocationCoords] = useState({ lat: 39.7392, lon: -104.9903 });
  const [locationStatus, setLocationStatus] = useState("resolved");

  const [weather, setWeather] = useState(null);
  const [weatherLoading, setWeatherLoading] = useState(false);
  const [weatherError, setWeatherError] = useState(false);
  const [selectedDate, setSelectedDate] = useState("today");
  const [customDate, setCustomDate] = useState("");

  const today = new Date();
  const todayStr = today.toISOString().split("T")[0];
  const maxDate = new Date(today); maxDate.setDate(today.getDate() + 7);
  const maxDateStr = maxDate.toISOString().split("T")[0];

  useEffect(() => { fetchWeather(39.7392, -104.9903, "Denver, Colorado, US", "today", ""); }, []);

  const fetchWeather = async (lat, lon, label, dateMode, date) => {
    setWeatherLoading(true); setWeather(null); setWeatherError(false);
    const isFuture = dateMode === "future" && !!date;
    try {
      const prompt = `Search for the ${isFuture ? `weather forecast for ${label} on ${date}` : `current weather in ${label} today`} and return ONLY this JSON (no markdown, no backticks):
{"max":75,"min":55,"code":1,"precip":10,"current":68}
Temps in Fahrenheit. code=WMO (0=clear,1=mostly clear,2=partly cloudy,3=overcast,61=rain,71=snow,80=showers,95=storm). precip=% chance 0-100. current=current temp or null if future.`;
      const text = await callClaude([{ role: "user", content: prompt }], 200, [{ type: "web_search_20250305", name: "web_search" }]);
      const parsed = extractJSON(text);
      setWeather({ max: parsed.max, min: parsed.min, code: parsed.code ?? 1, precip: parsed.precip ?? 0, current: parsed.current ?? null });
    } catch { setWeatherError(true); }
    setWeatherLoading(false);
  };

  const handleLocationSearch = async () => {
    if (!locationInput.trim()) return;
    setLocationStatus("searching"); setWeather(null);
    try {
      const text = await callClaude([{ role: "user", content: `GPS coordinates for "${locationInput}". Reply ONLY with JSON: {"lat":39.74,"lon":-104.99,"name":"Denver","country":"United States"}` }], 80);
      const parsed = extractJSON(text);
      if (!parsed.lat) throw new Error();
      const label = `${parsed.name}, ${parsed.country}`;
      setLocationCoords({ lat: parsed.lat, lon: parsed.lon });
      setLocationLabel(label); setLocationStatus("resolved");
      fetchWeather(parsed.lat, parsed.lon, label, selectedDate, customDate);
    } catch { setLocationStatus("error"); }
  };

  const handleDateChange = (val) => {
    setSelectedDate(val);
    if (val === "today") { setCustomDate(""); fetchWeather(locationCoords.lat, locationCoords.lon, locationLabel, "today", ""); }
  };

  const handleCustomDate = (val) => {
    setCustomDate(val);
    if (val) fetchWeather(locationCoords.lat, locationCoords.lon, locationLabel, "future", val);
  };

  const formatDate = (dateStr) => {
    if (!dateStr) return "";
    return new Date(dateStr + "T12:00:00").toLocaleDateString("en-US", { weekday: "long", month: "long", day: "numeric" });
  };

  const weatherSummary = () => {
    if (!weather) return "weather unknown";
    const cur = weather.current != null ? `, currently ${weather.current}°F` : "";
    return `${weatherDesc(weather.code)}, high ${weather.max}°F / low ${weather.min}°F${cur}, ${weather.precip}% chance of precipitation`;
  };

  const canGenerate = activity.trim() && (selectedDate === "today" || customDate) && locationStatus === "resolved";

  const generate = async () => {
    if (!canGenerate) return;
    setStep("loading"); setError(null);
    const dateLabel = selectedDate === "today" ? "today" : formatDate(customDate);

    setLoadingMsg("Building your look…");
    const prompt = `You are an expert personal stylist working in 2026. You MUST recommend only current, on-trend styles.

${TRENDS}

Client profile:
- Height: ${PROFILE.height}, inseam ${PROFILE.inseam}
- Waist: ${PROFILE.waist}, hips: ${PROFILE.hips}, chest: ${PROFILE.chest}
- Body shape: ${PROFILE.bodyShape}
- Coloring: ${PROFILE.coloring}
- Color guidance: ${PROFILE.colorNotes}
- Style: ${PROFILE.styleProfile}
- Fit notes: ${PROFILE.fitNotes}

Activity: ${activity} (${dateLabel}) in ${locationLabel}
Weather: ${weatherSummary()}
Extra notes: ${notes || "none"}

Rules:
- Incorporate collars and/or vests whenever natural for the occasion
- Boxy cropped tops are welcome — always pair with high-waisted bottoms
- Suggest specific pieces a Reformation/Anthropologie/Everlane/Abercrombie/Free People shopper would actually buy
- Warm muted palette only — no stark white, black, or cool tones

Reply ONLY with this JSON (no markdown, no backticks):
{
  "vibe": "2-3 word aesthetic label",
  "headline": "evocative outfit name, 5-7 words",
  "occasion": "one line on why this works for the activity",
  "trendNote": "one sentence on 2026 trends this taps",
  "pieces": [
    {"piece":"Top","desc":"specific description including silhouette and fabric","color":"color name"},
    {"piece":"Bottom","desc":"specific description","color":"color name"},
    {"piece":"Layer","desc":"specific description","color":"color name"},
    {"piece":"Shoes","desc":"specific description","color":"color name"},
    {"piece":"Accessories","desc":"specific description","color":"color name"}
  ],
  "colorStory": "2-sentence description of how the palette works together",
  "stylistTip": "one specific, actionable tip for wearing this look",
  "colorPalette": ["name1","name2","name3","name4"],
  "brandInspo": ["Brand: specific piece or section to shop"]
}`;

    try {
      const text = await callClaude([{ role: "user", content: prompt }], 1200);
      const outfit = extractJSON(text);
      setResult({ ...outfit, dateLabel, locationLabel, weatherSnap: weatherSummary() });
      setStep("result");
    } catch {
      setError("Something went wrong — try again.");
      setStep("form");
    }
  };

  const inputStyle = {
    width: "100%", padding: "10px 12px", borderRadius: "var(--border-radius-md)",
    border: "0.5px solid var(--color-border-secondary)", background: "var(--color-background-secondary)",
    color: "var(--color-text-primary)", fontSize: 14, boxSizing: "border-box", outline: "none",
    fontFamily: "var(--font-sans)"
  };

  const btnPrimary = (disabled) => ({
    width: "100%", padding: "11px", borderRadius: "var(--border-radius-md)", border: "none",
    background: disabled ? "var(--color-background-tertiary)" : "var(--color-text-primary)",
    color: disabled ? "var(--color-text-tertiary)" : "var(--color-background-primary)",
    fontSize: 14, fontWeight: 500, cursor: disabled ? "default" : "pointer", fontFamily: "var(--font-sans)"
  });

  if (step === "loading") return (
    <div style={{ minHeight: 320, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", gap: 16, padding: 40 }}>
      <div style={{ width: 36, height: 36, border: "1.5px solid var(--color-border-secondary)", borderTopColor: "var(--color-text-primary)", borderRadius: "50%", animation: "spin 0.8s linear infinite" }} />
      <style>{`@keyframes spin{to{transform:rotate(360deg)}}`}</style>
      <div style={{ fontSize: 14, color: "var(--color-text-secondary)" }}>{loadingMsg}</div>
    </div>
  );

  if (step === "result" && result) return (
    <div style={{ padding: "28px 24px", maxWidth: 560, margin: "0 auto" }}>

      {/* Header */}
      <div style={{ marginBottom: 24 }}>
        <div style={{ display: "flex", gap: 6, flexWrap: "wrap", marginBottom: 14 }}>
          <Tag accent>{result.vibe}</Tag>
          <Tag><i className="ti ti-map-pin" style={{ fontSize: 11, marginRight: 4 }} aria-hidden />{result.locationLabel}</Tag>
          <Tag><i className="ti ti-calendar" style={{ fontSize: 11, marginRight: 4 }} aria-hidden />{result.dateLabel}</Tag>
        </div>
        <div style={{ fontSize: 22, fontWeight: 500, color: "var(--color-text-primary)", lineHeight: 1.25, marginBottom: 10 }}>{result.headline}</div>
        <div style={{ fontSize: 13, color: "var(--color-text-tertiary)", display: "flex", alignItems: "center", gap: 6, marginBottom: 12 }}>
          <i className={`ti ${weatherIcon(0)}`} style={{ fontSize: 14 }} aria-hidden />
          {result.weatherSnap}
        </div>
        <div style={{ fontSize: 14, color: "var(--color-text-secondary)", lineHeight: 1.65, marginBottom: 6 }}>{result.occasion}</div>
        {result.trendNote && <div style={{ fontSize: 13, color: "var(--color-text-tertiary)", fontStyle: "italic", lineHeight: 1.5 }}>{result.trendNote}</div>}
      </div>

      <Divider />

      {/* Color story */}
      {result.colorPalette && (
        <div style={{ marginBottom: 24 }}>
          <Label>Color palette</Label>
          <div style={{ display: "flex", gap: 8, flexWrap: "wrap", marginBottom: 10 }}>
            {result.colorPalette.map((c, i) => (
              <span key={i} style={{ fontSize: 13, color: "var(--color-text-secondary)", background: "var(--color-background-tertiary)", padding: "5px 13px", borderRadius: 20, border: "0.5px solid var(--color-border-tertiary)" }}>{c}</span>
            ))}
          </div>
          {result.colorStory && <div style={{ fontSize: 13, color: "var(--color-text-tertiary)", lineHeight: 1.6 }}>{result.colorStory}</div>}
        </div>
      )}

      <Divider />

      {/* Pieces */}
      <div style={{ marginBottom: 24 }}>
        <Label>The outfit</Label>
        <div style={{ display: "flex", flexDirection: "column", gap: 1 }}>
          {result.pieces?.map((p, i) => (
            <div key={i} style={{ padding: "14px 0", borderBottom: i < result.pieces.length - 1 ? "0.5px solid var(--color-border-tertiary)" : "none" }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", gap: 12, marginBottom: 4 }}>
                <span style={{ fontSize: 13, fontWeight: 500, color: "var(--color-text-primary)", textTransform: "uppercase", letterSpacing: "0.06em" }}>{p.piece}</span>
                {p.color && <span style={{ fontSize: 12, color: "var(--color-text-tertiary)", whiteSpace: "nowrap", flexShrink: 0 }}>{p.color}</span>}
              </div>
              <div style={{ fontSize: 14, color: "var(--color-text-secondary)", lineHeight: 1.55 }}>{p.desc}</div>
            </div>
          ))}
        </div>
      </div>

      <Divider />

      {/* Stylist tip */}
      {result.stylistTip && (
        <div style={{ marginBottom: 24 }}>
          <Label>Stylist tip</Label>
          <div style={{ fontSize: 14, color: "var(--color-text-secondary)", lineHeight: 1.65, paddingLeft: 12, borderLeft: "2px solid var(--color-border-secondary)" }}>{result.stylistTip}</div>
        </div>
      )}

      {/* Shop */}
      {result.brandInspo?.length > 0 && (
        <>
          <Divider />
          <div style={{ marginBottom: 24 }}>
            <Label>Shop the look</Label>
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {result.brandInspo.map((b, i) => (
                <div key={i} style={{ fontSize: 13, color: "var(--color-text-secondary)", display: "flex", alignItems: "flex-start", gap: 8 }}>
                  <i className="ti ti-arrow-right" style={{ fontSize: 14, color: "var(--color-text-tertiary)", marginTop: 1, flexShrink: 0 }} aria-hidden />
                  {b}
                </div>
              ))}
            </div>
          </div>
        </>
      )}

      <Divider />

      {/* Actions */}
      <div style={{ display: "flex", gap: 10 }}>
        <button onClick={() => { setActivity(""); setNotes(""); setResult(null); setStep("form"); }}
          style={{ flex: 1, padding: "10px", borderRadius: "var(--border-radius-md)", border: "0.5px solid var(--color-border-secondary)", background: "var(--color-background-secondary)", color: "var(--color-text-primary)", fontSize: 14, cursor: "pointer", fontFamily: "var(--font-sans)" }}>
          New activity
        </button>
        <button onClick={generate}
          style={{ flex: 1, padding: "10px", borderRadius: "var(--border-radius-md)", border: "none", background: "var(--color-text-primary)", color: "var(--color-background-primary)", fontSize: 14, fontWeight: 500, cursor: "pointer", fontFamily: "var(--font-sans)" }}>
          Try another look
        </button>
      </div>
    </div>
  );

  return (
    <div style={{ padding: "28px 24px", maxWidth: 540, margin: "0 auto" }}>

      {/* Header */}
      <div style={{ marginBottom: 24 }}>
        <div style={{ fontSize: 20, fontWeight: 500, color: "var(--color-text-primary)", marginBottom: 4 }}>What to wear</div>
        <div style={{ fontSize: 14, color: "var(--color-text-secondary)", lineHeight: 1.5, marginBottom: 14 }}>Styled for your measurements, coloring, and weather.</div>
        <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
          {["5'6.5\"", "Soft Autumn", "Romantic minimalist", "Loves collars & vests"].map(t => <Tag key={t}>{t}</Tag>)}
        </div>
      </div>

      <Divider />

      {error && <div style={{ color: "var(--color-text-danger)", fontSize: 13, marginBottom: 16 }}>{error}</div>}

      {/* Location */}
      <div style={{ marginBottom: 20 }}>
        <Label>Location</Label>
        <div style={{ display: "flex", gap: 8 }}>
          <input value={locationInput} onChange={e => { setLocationInput(e.target.value); setLocationStatus("idle"); }}
            onKeyDown={e => e.key === "Enter" && handleLocationSearch()}
            placeholder="City, state or country"
            style={{ ...inputStyle, flex: 1 }} />
          <button onClick={handleLocationSearch}
            style={{ padding: "10px 16px", borderRadius: "var(--border-radius-md)", border: "0.5px solid var(--color-border-secondary)", background: "var(--color-background-secondary)", color: "var(--color-text-primary)", fontSize: 13, cursor: "pointer", whiteSpace: "nowrap", fontFamily: "var(--font-sans)" }}>
            {locationStatus === "searching" ? "…" : "Update"}
          </button>
        </div>
        {locationStatus === "error" && <div style={{ fontSize: 12, color: "var(--color-text-danger)", marginTop: 5 }}>City not found — try again.</div>}
        {locationStatus === "resolved" && <div style={{ fontSize: 12, color: "var(--color-text-tertiary)", marginTop: 5 }}><i className="ti ti-map-pin" style={{ fontSize: 12, marginRight: 4 }} aria-hidden />{locationLabel}</div>}
      </div>

      {/* When */}
      <div style={{ marginBottom: 20 }}>
        <Label>When</Label>
        <div style={{ display: "flex", gap: 8, marginBottom: 10 }}>
          {["today", "future"].map(opt => (
            <button key={opt} onClick={() => handleDateChange(opt)}
              style={{ padding: "7px 18px", borderRadius: 20, border: `0.5px solid ${selectedDate === opt ? "var(--color-text-primary)" : "var(--color-border-secondary)"}`, background: selectedDate === opt ? "var(--color-text-primary)" : "transparent", color: selectedDate === opt ? "var(--color-background-primary)" : "var(--color-text-secondary)", fontSize: 13, cursor: "pointer", fontWeight: selectedDate === opt ? 500 : 400, fontFamily: "var(--font-sans)" }}>
              {opt === "today" ? "Today" : "Another day"}
            </button>
          ))}
        </div>
        {selectedDate === "future" && (
          <input type="date" min={todayStr} max={maxDateStr} value={customDate}
            onChange={e => handleCustomDate(e.target.value)}
            style={{ ...inputStyle, marginBottom: 10 }} />
        )}
        <div style={{ padding: "10px 14px", borderRadius: "var(--border-radius-md)", background: "var(--color-background-tertiary)", border: "0.5px solid var(--color-border-tertiary)", fontSize: 13, color: "var(--color-text-secondary)", display: "flex", alignItems: "center", gap: 8 }}>
          {weatherLoading
            ? <><i className="ti ti-loader" style={{ fontSize: 14 }} aria-hidden /> Fetching forecast…</>
            : weatherError
              ? <><i className="ti ti-alert-circle" style={{ fontSize: 14 }} aria-hidden /> Weather unavailable</>
              : weather
                ? <><i className={`ti ${weatherIcon(weather.code)}`} style={{ fontSize: 14 }} aria-hidden /> {weatherDesc(weather.code)} · {weather.max}° / {weather.min}°F{weather.current != null ? ` · now ${weather.current}°F` : ""} · {weather.precip}% precip</>
                : selectedDate === "future" && !customDate
                  ? "Pick a date to see the forecast"
                  : "Loading…"}
        </div>
      </div>

      {/* Activity */}
      <div style={{ marginBottom: 20 }}>
        <Label>Activity</Label>
        <input value={activity} onChange={e => setActivity(e.target.value)}
          onKeyDown={e => e.key === "Enter" && generate()}
          placeholder="e.g. ski day, rooftop dinner, farmers market, job interview…"
          style={{ ...inputStyle, marginBottom: 8 }} />
        <input value={notes} onChange={e => setNotes(e.target.value)}
          placeholder="Anything else? e.g. want to wear a dress, no heels…"
          style={inputStyle} />
      </div>

      <button onClick={generate} disabled={!canGenerate} style={btnPrimary(!canGenerate)}>
        Style me
      </button>
    </div>
  );
}
