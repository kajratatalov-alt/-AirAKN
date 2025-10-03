# -AirAKN
Все для тебя! Учи РКЭ и не будешь жаловаться ❤️
{
  "name": "airakn-quiz",
  "private": true,
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview --port 5173"
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.1",
    "vite": "^5.4.8",
    "vite-plugin-pwa": "^0.20.0"
  }
}import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { VitePWA } from "vite-plugin-pwa";

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: "autoUpdate",
      includeAssets: ["favicon.svg", "icons/*.png"],
      manifest: {
        name: "AirAKN",
        short_name: "AirAKN",
        description: "Учебная викторина по Руководству кабинного экипажа",
        theme_color: "#0d2540",
        background_color: "#072033",
        display: "standalone",
        start_url: "/",
        icons: [
          { src: "/icons/icon-192.png", sizes: "192x192", type: "image/png" },
          { src: "/icons/icon-512.png", sizes: "512x512", type: "image/png" },
          { src: "/icons/maskable-512.png", sizes: "512x512", type: "image/png", purpose: "maskable" }
        ]
      }
    })
  ]
});<!DOCTYPE html>
<html lang="ru">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#0d2540" />
    <link rel="manifest" href="/manifest.webmanifest" />
    <link rel="icon" href="/favicon.svg" />
    <title>AirAKN</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>[
  {
    "question_ru": "Сколько аптечек должно быть на борту?",
    "question_en": "How many first aid kits must be on board?",
    "options_ru": ["1", "2", "3", "4"],
    "options_en": ["1", "2", "3", "4"],
    "answer": 2
  },
  {
    "question_ru": "Какая команда подаётся при разгерметизации?",
    "question_en": "Which command is given during decompression?",
    "options_ru": ["Одевайте кислородные маски!", "Застегните ремни!", "Принять позу для удара!", "Эвакуация!"],
    "options_en": ["Put on oxygen masks!", "Fasten seatbelts!", "Brace!", "Evacuate!"],
    "answer": 0
  }
]import React, { useState, useEffect } from "react";

const STORAGE_KEY = "airakn-quiz-questions";

export default function App() {
  const [questions, setQuestions] = useState([]);
  const [index, setIndex] = useState(0);
  const [selected, setSelected] = useState(null);
  const [score, setScore] = useState(0);
  const [lang, setLang] = useState("ru");

  useEffect(() => {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) setQuestions(JSON.parse(stored));
    else
      fetch("/questions.json")
        .then(r => r.json())
        .then(js => {
          setQuestions(js);
          localStorage.setItem(STORAGE_KEY, JSON.stringify(js));
        })
        .catch(() => setQuestions([]));
  }, []);

  if (!questions.length) return <p>Загрузка вопросов...</p>;

  const current = questions[index];

  function handleAnswer(i) {
    setSelected(i);
    if (i === current.answer) setScore(score + 1);
  }

  function next() {
    setSelected(null);
    setIndex(index + 1);
  }

  function restart() {
    setIndex(0);
    setScore(0);
    setSelected(null);
  }

  return (
    <div className="app">
      <h1>AirAKN</h1>

      <div style={{ marginBottom: "10px" }}>
        <button onClick={() => setLang("ru")}>RU</button>
        <button onClick={() => setLang("en")}>EN</button>
      </div>

      {index < questions.length ? (
        <>
          <p>Вопрос {index + 1} из {questions.length}</p>
          <h2>{lang === "ru" ? current.question_ru : current.question_en}</h2>

          <div className="options">
            {(lang === "ru" ? current.options_ru : current.options_en).map((opt, i) => (
              <button
                key={i}
                onClick={() => handleAnswer(i)}
                disabled={selected !== null}
                className={
                  selected === null ? "" :
                  i === current.answer ? "correct" :
                  i === selected ? "wrong" : ""
                }
              >
                {opt}
              </button>
            ))}
          </div>

          {selected !== null && index < questions.length - 1 && (
            <button className="next" onClick={next}>Следующий</button>
          )}
          {selected !== null && index === questions.length - 1 && (
            <>
              <h2>Игра окончена! Ваш результат: {score}/{questions.length}</h2>
              <button onClick={restart}>Начать заново</button>
            </>
          )}
        </>
      ) : null}
    </div>
  );
}import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App";
import "./styles.css";
import { registerSW } from "./registerServiceWorker";

createRoot(document.getElementById("root")).render(<App />);
registerSW();export function registerSW() {
  if ("serviceWorker" in navigator) {
    window.addEventListener("load", async () => {
      try {
        const registration = await navigator.serviceWorker.register("/sw.js");
        console.log("SW registered:", registration);
      } catch (err) {
        console.error("SW failed:", err);
      }
    });
  }
}body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f0f2f5;
  color: #222;
  text-align: center;
}

.app {
  padding: 20px;
  max-width: 600px;
  margin: auto;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin: 20px 0;
}

button {
  padding: 10px;
  font-size: 16px;
  cursor: pointer;
}

button.correct {
  background: #4caf50;
  color: white;
}

button.wrong {
  background: #f44336;
  color: white;
}

.next {
  margin-top: 20px;
  padding: 10px 20px;
}

button:disabled {
  opacity: 0.7;
  cursor: default;
}