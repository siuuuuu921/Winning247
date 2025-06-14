// src/App.js
import React, { useState } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Home from './Home';
import MinesGame from './MinesGame';

function App() {
  const [user, setUser] = useState({
    bonusBalance: 200,
    realBalance: 0,
    totalMinesRounds: 0,
  });

  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route
          path="/mines"
          element={<MinesGame user={user} setUser={setUser} />}
        />
      </Routes>
    </Router>
  );
}

export default App;


// src/Home.jsx
import React from 'react';
import { Link } from 'react-router-dom';

const Home = () => {
  return (
    <div className="p-6 text-center bg-gray-900 text-white min-h-screen">
      <h1 className="text-3xl font-bold mb-4">🎰 Real Money Betting</h1>
      <p className="mb-4">Welcome! You have ₹200 bonus to use in Mines.</p>
      <Link to="/mines">
        <button className="bg-yellow-400 text-black px-4 py-2 rounded-xl shadow-xl hover:bg-yellow-300">
          Play Mines
        </button>
      </Link>
    </div>
  );
};

export default Home;


// src/MinesGame.jsx
import React, { useState, useEffect } from 'react';
import './MinesGame.css';

const generateBoard = (mineCount = 3) => {
  const size = 25;
  const mines = new Set();
  while (mines.size < mineCount) {
    mines.add(Math.floor(Math.random() * size));
  }
  return [...Array(size)].map((_, i) => ({
    id: i,
    isMine: mines.has(i),
    revealed: false,
  }));
};

const shouldWinRound = (roundNumber) => {
  const rand = Math.random() * 10;
  return roundNumber < 10 ? rand < 7 : rand < 4;
};

const MinesGame = ({ user, setUser }) => {
  const [board, setBoard] = useState(generateBoard());
  const [gameOver, setGameOver] = useState(false);
  const [winCount, setWinCount] = useState(0);

  const clickTile = (tileId) => {
    if (gameOver || board[tileId].revealed) return;

    const newBoard = [...board];
    const win = shouldWinRound(user.totalMinesRounds);
    newBoard[tileId].revealed = true;
    newBoard[tileId].isMine = !win;

    if (!win) {
      setGameOver(true);
      setUser({ ...user, totalMinesRounds: user.totalMinesRounds + 1 });
    } else {
      setWinCount(winCount + 1);
    }

    setBoard(newBoard);
  };

  const cashOut = () => {
    const realEarned = winCount * 10;
    setUser({
      ...user,
      bonusBalance: user.bonusBalance,
      realBalance: user.realBalance + realEarned,
      totalMinesRounds: user.totalMinesRounds + 1,
    });
    resetGame();
  };

  const resetGame = () => {
    setBoard(generateBoard());
    setGameOver(false);
    setWinCount(0);
  };

  return (
    <div className="mines-container">
      <h2>Mines Game</h2>
      <p>Bonus Balance: ₹{user.bonusBalance}</p>
      <p>Real Balance: ₹{user.realBalance}</p>
      <div className="grid">
        {board.map((tile) => (
          <button
            key={tile.id}
            className={`tile ${tile.revealed ? (tile.isMine ? 'mine' : 'safe') : ''}`}
            onClick={() => clickTile(tile.id)}
            disabled={tile.revealed || gameOver}
          >
            {tile.revealed ? (tile.isMine ? '💣' : '💎') : ''}
          </button>
        ))}
      </div>
      <div className="controls">
        <p>Current Round Winnings: ₹{winCount * 10}</p>
        {gameOver ? (
          <p className="lost">💥 You hit a mine! Game Over.</p>
        ) : (
          <button onClick={cashOut} className="cashout">💰 Cash Out</button>
        )}
        <button onClick={resetGame} className="reset">🔁 Reset</button>
      </div>
    </div>
  );
};

export default MinesGame;


// src/MinesGame.css
.mines-container {
  background: #1f1f1f;
  color: #fff;
  padding: 20px;
  text-align: center;
  min-height: 100vh;
}
.grid {
  display: grid;
  grid-template-columns: repeat(5, 60px);
  gap: 10px;
  margin: 20px auto;
  justify-content: center;
}
.tile {
  width: 60px;
  height: 60px;
  font-size: 24px;
  background: #2e2e2e;
  border: 2px solid #444;
  border-radius: 8px;
  cursor: pointer;
}
.tile.safe {
  background: #10b981;
}
.tile.mine {
  background: #ef4444;
}
.controls {
  margin-top: 20px;
}
.cashout {
  background: #facc15;
  color: #000;
  padding: 8px 14px;
  border: none;
  border-radius: 8px;
  margin: 5px;
}
.reset {
  background: #3b82f6;
  color: #fff;
  padding: 8px 14px;
  border: none;
  border-radius: 8px;
}
.lost {
  color: red;
  font-weight: bold;
}
