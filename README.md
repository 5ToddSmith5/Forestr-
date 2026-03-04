# Forestr-
A simple Tic Tac Toe Game
import { useState, useEffect, useCallback } from "react";

const winPatterns = [
  [0, 1, 2], [3, 4, 5], [6, 7, 8],
  [0, 3, 6], [1, 4, 7], [2, 5, 8],
  [0, 4, 8], [2, 4, 6],
];

function findBestMove(board, player) {
  for (let pattern of winPatterns) {
    const values = pattern.map(i => board[i]);
    const playerCount = values.filter(v => v === player).length;
    const emptyCount = values.filter(v => v === "").length;
    if (playerCount === 2 && emptyCount === 1) {
      return pattern.find(i => board[i] === "");
    }
  }
  return -1;
}

function checkWin(board, player) {
  return winPatterns.some(pattern => pattern.every(i => board[i] === player));
}

function getWinningPattern(board, player) {
  return winPatterns.find(pattern => pattern.every(i => board[i] === player));
}

export default function ForestTicTacToe() {
  const [board, setBoard] = useState(Array(9).fill(""));
  const [currentPlayer, setCurrentPlayer] = useState("X");
  const [gameActive, setGameActive] = useState(true);
  const [gameMode, setGameMode] = useState("pvp");
  const [scores, setScores] = useState({ X: 0, O: 0 });
  const [winnerCells, setWinnerCells] = useState([]);
  const [statusText, setStatusText] = useState("🌲 Tree's turn");
  const [isWinner, setIsWinner] = useState(false);

  const endGame = useCallback((winner, newBoard) => {
    setGameActive(false);
    if (winner) {
      const pattern = getWinningPattern(newBoard, winner);
      setWinnerCells(pattern);
      setScores(prev => ({ ...prev, [winner]: prev[winner] + 1 }));
      const symbol = winner === "X" ? "🌲 Tree" : "🍄 Mushroom";
      setStatusText(`${symbol} wins! 🎉`);
      setIsWinner(true);
    } else {
      setStatusText("🤝 It's a tie!");
    }
  }, []);

  const makeMove = useCallback((index, player, currentBoard) => {
    const newBoard = [...currentBoard];
    newBoard[index] = player;
    setBoard(newBoard);

    if (checkWin(newBoard, player)) {
      endGame(player, newBoard);
      return { board: newBoard, ended: true };
    } else if (newBoard.every(c => c !== "")) {
      endGame(null, newBoard);
      return { board: newBoard, ended: true };
    } else {
      const next = player === "X" ? "O" : "X";
      setCurrentPlayer(next);
      const symbol = next === "X" ? "🌲 Tree" : "🍄 Mushroom";
      setStatusText(`${symbol}'s turn`);
      return { board: newBoard, ended: false };
    }
  }, [endGame]);

  const handleClick = useCallback((index) => {
    if (board[index] !== "" || !gameActive) return;
    if (gameMode === "ai" && currentPlayer === "O") return;
    makeMove(index, currentPlayer, board);
  }, [board, gameActive, gameMode, currentPlayer, makeMove]);

  // AI move effect
  useEffect(() => {
    if (gameMode !== "ai" || currentPlayer !== "O" || !gameActive) return;
    const timer = setTimeout(() => {
      let move = findBestMove(board, "O");
      if (move === -1) move = findBestMove(board, "X");
      if (move === -1 && board[4] === "") move = 4;
      if (move === -1) {
        const corners = [0, 2, 6, 8].filter(i => board[i] === "");
        if (corners.length > 0) move = corners[Math.floor(Math.random() * corners.length)];
      }
      if (move === -1) {
        const available = board.map((v, i) => v === "" ? i : -1).filter(i => i !== -1);
        if (available.length > 0) move = available[Math.floor(Math.random() * available.length)];
      }
      if (move !== -1) makeMove(move, "O", board);
    }, 600);
    return () => clearTimeout(timer);
  }, [gameMode, currentPlayer, gameActive, board, makeMove]);

  const resetGame = () => {
    setBoard(Array(9).fill(""));
    setCurrentPlayer("X");
    setGameActive(true);
    setWinnerCells([]);
    setStatusText("🌲 Tree's turn");
    setIsWinner(false);
  };

  const handleSetMode = (mode) => {
    setGameMode(mode);
    setBoard(Array(9).fill(""));
    setCurrentPlayer("X");
    setGameActive(true);
    setWinnerCells([]);
    setStatusText("🌲 Tree's turn");
    setIsWinner(false);
  };

  return (
    <div style={styles.body}>
      {/* Fireflies */}
      {Array.from({ length: 20 }).map((_, i) => (
        <div key={i} style={{
          ...styles.firefly,
          left: `${(i * 37 + 13) % 100}%`,
          top: `${(i * 53 + 7) % 100}%`,
          animationDelay: `${(i * 0.3) % 6}s`,
        }} />
      ))}

      <div style={styles.container}>
        <h1 style={styles.h1}>🌲 Forest Tic-Tac-Toe 🌲</h1>
        <p style={styles.subtitle}>🌿 Where nature plays! 🍄</p>

        {/* Mode Toggle */}
        <div style={styles.modeToggle}>
          <button
            style={{ ...styles.modeBtn, ...(gameMode === "pvp" ? styles.modeBtnActive : {}) }}
            onClick={() => handleSetMode("pvp")}
          >👥 2 Players</button>
          <button
            style={{ ...styles.modeBtn, ...(gameMode === "ai" ? styles.modeBtnActive : {}) }}
            onClick={() => handleSetMode("ai")}
          >🤖 vs Forest Spirit</button>
        </div>

        {/* Scoreboard */}
        <div style={styles.scoreboard}>
          <div style={{ ...styles.scoreCard, borderColor: "#81C784" }}>
            <div style={{ fontSize: 28 }}>🌲</div>
            <div style={styles.scoreLabel}>Tree</div>
            <div style={styles.scoreValue}>{scores.X}</div>
          </div>
          <div style={{ ...styles.scoreCard, borderColor: "#EF9A9A" }}>
            <div style={{ fontSize: 28 }}>🍄</div>
            <div style={styles.scoreLabel}>Mushroom</div>
            <div style={styles.scoreValue}>{scores.O}</div>
          </div>
        </div>

        {/* Board */}
        <div style={styles.gameWrapper}>
          <div style={styles.boardGrid}>
            {board.map((cell, i) => {
              const isWinCell = winnerCells.includes(i);
              return (
                <button
                  key={i}
                  onClick={() => handleClick(i)}
                  style={{
                    ...styles.cell,
                    ...(isWinCell ? styles.cellWinner : {}),
                    cursor: cell !== "" || !gameActive ? "not-allowed" : "pointer",
                  }}
                >
                  {cell === "X" && <span style={{ fontSize: 44 }}>🌲</span>}
                  {cell === "O" && <span style={{ fontSize: 44 }}>🍄</span>}
                </button>
              );
            })}
          </div>
        </div>

        {/* Status */}
        <div style={{ ...styles.status, ...(isWinner ? styles.statusWinner : {}) }}>
          {statusText}
        </div>

        {/* Reset */}
        <button style={styles.btn} onClick={resetGame}>🔄 New Game</button>
      </div>

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fredoka+One&family=Nunito:wght@400;700&display=swap');
        @keyframes glow {
          0%, 100% { opacity: 0.2; }
          50% { opacity: 1; }
        }
        @keyframes float {
          0%, 100% { transform: translate(0, 0); }
          25% { transform: translate(30px, -30px); }
          50% { transform: translate(-20px, -50px); }
          75% { transform: translate(-30px, -20px); }
        }
        @keyframes pulse {
          0%, 100% { transform: scale(1); }
          50% { transform: scale(1.03); }
        }
        @keyframes winnerPulse {
          0%, 100% { transform: scale(1); background: linear-gradient(145deg,#FFD54F,#FFC107); }
          50% { transform: scale(1.08); background: linear-gradient(145deg,#FFC107,#FFD54F); }
        }
      `}</style>
    </div>
  );
}

const styles = {
  body: {
    minHeight: "100vh",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
    fontFamily: "'Nunito', sans-serif",
    background: "linear-gradient(180deg, #87CEEB 0%, #98D4BB 20%, #2D5A27 40%, #1E3D1A 100%)",
    overflow: "hidden",
    position: "relative",
    flexDirection: "column",
  },
  firefly: {
    position: "fixed",
    width: 6,
    height: 6,
    background: "#FFE66D",
    borderRadius: "50%",
    boxShadow: "0 0 10px #FFE66D, 0 0 20px #FFE66D",
    animation: "glow 2s ease-in-out infinite, float 6s ease-in-out infinite",
    pointerEvents: "none",
    zIndex: 0,
  },
  container: {
    position: "relative",
    zIndex: 10,
    textAlign: "center",
    padding: 20,
    display: "flex",
    flexDirection: "column",
    alignItems: "center",
  },
  h1: {
    fontFamily: "'Fredoka One', cursive",
    fontSize: "2.4em",
    color: "#fff",
    textShadow: "3px 3px 0 #2D5A27, 5px 5px 10px rgba(0,0,0,0.3)",
    marginBottom: 8,
  },
  subtitle: {
    fontSize: "1.1em",
    color: "#C8E6C9",
    marginBottom: 16,
    textShadow: "1px 1px 2px rgba(0,0,0,0.3)",
  },
  modeToggle: { marginBottom: 16, display: "flex", gap: 8 },
  modeBtn: {
    padding: "10px 22px",
    fontFamily: "'Nunito', sans-serif",
    fontWeight: 700,
    fontSize: "0.95em",
    color: "#fff",
    background: "rgba(255,255,255,0.2)",
    border: "2px solid rgba(255,255,255,0.3)",
    borderRadius: 20,
    cursor: "pointer",
  },
  modeBtnActive: {
    background: "linear-gradient(145deg, #4CAF50, #388E3C)",
    borderColor: "#4CAF50",
  },
  scoreboard: { display: "flex", gap: 24, marginBottom: 16 },
  scoreCard: {
    background: "rgba(255,255,255,0.15)",
    padding: "12px 22px",
    borderRadius: 14,
    border: "2px solid",
    minWidth: 110,
    backdropFilter: "blur(8px)",
  },
  scoreLabel: { color: "#fff", fontWeight: 700, fontSize: "0.95em" },
  scoreValue: { fontSize: "1.8em", fontWeight: 700, color: "#FFD54F" },
  gameWrapper: {
    background: "linear-gradient(145deg, #8B4513, #654321)",
    padding: 18,
    borderRadius: 20,
    boxShadow: "0 10px 40px rgba(0,0,0,0.4)",
    border: "4px solid #3E2723",
  },
  boardGrid: {
    display: "grid",
    gridTemplateColumns: "repeat(3, 95px)",
    gridTemplateRows: "repeat(3, 95px)",
    gap: 8,
    background: "linear-gradient(145deg, #5D4037, #4E342E)",
    padding: 10,
    borderRadius: 14,
  },
  cell: {
    width: 95,
    height: 95,
    background: "linear-gradient(145deg, #A5D6A7, #81C784)",
    border: "none",
    borderRadius: 12,
    fontSize: 48,
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
    transition: "all 0.2s ease",
    boxShadow: "0 4px 8px rgba(0,0,0,0.2)",
  },
  cellWinner: {
    background: "linear-gradient(145deg, #FFD54F, #FFC107)",
    animation: "winnerPulse 0.6s ease-in-out infinite",
  },
  status: {
    marginTop: 18,
    fontSize: "1.3em",
    fontWeight: 700,
    color: "#fff",
    padding: "14px 28px",
    background: "linear-gradient(145deg, #4CAF50, #388E3C)",
    borderRadius: 30,
    boxShadow: "0 4px 15px rgba(0,0,0,0.3)",
    minWidth: 260,
    display: "inline-block",
  },
  statusWinner: {
    background: "linear-gradient(145deg, #FFD54F, #FFA000)",
    animation: "pulse 1s ease-in-out infinite",
  },
  btn: {
    marginTop: 18,
    padding: "13px 38px",
    fontFamily: "'Fredoka One', cursive",
    fontSize: "1.15em",
    color: "#fff",
    background: "linear-gradient(145deg, #FF7043, #E64A19)",
    border: "none",
    borderRadius: 30,
    cursor: "pointer",
    boxShadow: "0 4px 15px rgba(0,0,0,0.3), 0 6px 0 #BF360C",
  },
};
