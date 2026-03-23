C:\Users\YourName\Desktop\Trading-Bot# Start bot
curl -X POST http://localhost:3000/start

# Check status
curl http://localhost:3000/status

# View history
curl http://localhost:3000/historyPOST http://localhost:3000/stopGET http://localhost:3000/statusPOST http://localhost:3000/startnpm startAPI_KEY=your_actual_api_key_here
SECRET_KEY=your_actual_secret_key_herenpm installgit clone https://github.com/mndebelecollen074-netizen/Trading-bot-.git
cd Trading-bot-npm install
npm startconst assert = require('assert');

const { analyze, calculateQuantity, detectMarket, tradeManagement } = require('./tradingBot');

describe('Trading Bot Functions', () => {
  describe('analyze', () => {
    it('should return the correct analysis for given market data', () => {
      // Example market data
      const data = { price: 100, volume: 2000 };
      const result = analyze(data);
      assert.strictEqual(result, 'expectedAnalysis'); // Replace with expected value
    });
  });

  describe('calculateQuantity', () => {
    it('should calculate the correct quantity based on risk', () => {
      const risk = 100;
      const balance = 1000;
      const quantity = calculateQuantity(risk, balance);
      assert.strictEqual(quantity, 10); // Replace with expected value
    });
  });

  describe('detectMarket', () => {
    it('should identify the correct market trend', () => {
      const marketData = [/* Sample market data */];
      const trend = detectMarket(marketData);
      assert.strictEqual(trend, 'bullish'); // Replace with expected value
    });
  });

  describe('tradeManagement', () => {
    it('should execute the trade management logic correctly', () => {
      const trade = { /* trade details */ };
      const result = tradeManagement(trade);
      assert.strictEqual(result.success, true); // Replace with expected condition
    });
  });
});
curl -X POST http://localhost:3000/stopcurl -X POST http://localhost:3000/startcurl http://localhost:3000/statusnpm test# Install dependencies
npm install

# Start the bot
npm startAPI_KEY=your_binance_api_key
SECRET_KEY=your_binance_secret_keynpm run dev# Trading-bot-const express = require("express");
const axios = require("axios");
const crypto = require("crypto");
const { RSI, EMA } = require("technicalindicators");

const app = express();
app.use(express.json());

// ===== CONFIG =====
const API_KEY = process.env.API_KEY;
const SECRET_KEY = process.env.SECRET_KEY;

const SYMBOL = "BTCUSDT";
const INTERVAL = "15m";

let botRunning = true;

// ===== RISK SETTINGS =====
const RISK_PERCENT = 0.01;
const ACCOUNT_SIZE = 100;

let activeTrade = null;
let tradesToday = 0;
let lossCount = 0;

// ===== LEARNING =====
let tradeHistory = [];
let strategyStats = {
  TREND: { wins: 0, losses: 0 },
  RANGE: { wins: 0, losses: 0 },
  BREAKOUT: { wins: 0, losses: 0 }
};

// ===== GET MARKET DATA =====
async function getMarketData() {
  const res = await axios.get(
    `https://api.binance.com/api/v3/klines?symbol=${SYMBOL}&interval=${INTERVAL}&limit=100`
  );

  return res.data.map(c => parseFloat(c[4]));
}

// ===== MARKET DETECTION =====
function detectMarket(ema50, ema200) {
  const diff = Math.abs(ema50 - ema200);
  if (diff < 10) return "SIDEWAYS";
  return "TRENDING";
}

// ===== STRATEGY =====
function analyze(closes) {
  const ema50 = EMA.calculate({ period: 50, values: closes });
  const ema200 = EMA.calculate({ period: 200, values: closes });
  const rsi = RSI.calculate({ period: 14, values: closes });

  const price = closes[closes.length - 1];
  const e50 = ema50[ema50.length - 1];
  const e200 = ema200[ema200.length - 1];
  const r = rsi[rsi.length - 1];

  let signal = "NO TRADE";

  if (e50 > e200 && r > 50 && r < 70 && price > e50) signal = "BUY";
  if (e50 < e200 && r < 50 && r > 30 && price < e50) signal = "SELL";

  return { signal, price, rsi: r, ema50: e50, ema200: e200 };
}

// ===== POSITION SIZE =====
function calculateQuantity(price) {
  const riskAmount = ACCOUNT_SIZE * RISK_PERCENT;
  const stopLossPercent = 0.02;
  return (riskAmount / (price * stopLossPercent)).toFixed(6);
}

// ===== PLACE ORDER =====
async function placeOrder(side, price) {
  const quantity = calculateQuantity(price);

  const stopLoss =
    side === "BUY" ? price * 0.98 : price * 1.02;

  const takeProfit =
    side === "BUY" ? price * 1.04 : price * 0.96;

  console.log("ORDER:", side, quantity);

  const timestamp = Date.now();

  const query = `symbol=${SYMBOL}&side=${side}&type=MARKET&quantity=${quantity}&timestamp=${timestamp}`;

  const signature = crypto
    .createHmac("sha256", SECRET_KEY)
    .update(query)
    .digest("hex");

  await axios.post(
    `https://api.binance.com/api/v3/order?${query}&signature=${signature}`,
    null,
    { headers: { "X-MBX-APIKEY": API_KEY } }
  );

  activeTrade = {
    side,
    entry: price,
    stopLoss,
    takeProfit,
    quantity
  };

  tradesToday++;
}

// ===== MANAGE TRADE =====
async function manageTrade(price) {
  if (!activeTrade) return;

  if (activeTrade.side === "BUY") {
    if (price > activeTrade.entry * 1.02) {
      activeTrade.stopLoss = price * 0.99;
    }

    if (price >= activeTrade.takeProfit || price <= activeTrade.stopLoss) {
      await closeTrade(price);
    }
  }

  if (activeTrade.side === "SELL") {
    if (price < activeTrade.entry * 0.98) {
      activeTrade.stopLoss = price * 1.01;
    }

    if (price <= activeTrade.takeProfit || price >= activeTrade.stopLoss) {
      await closeTrade(price);
    }
  }
}

// ===== CLOSE TRADE =====
async function closeTrade(price) {
  console.log("Closing trade");

  const side = activeTrade.side === "BUY" ? "SELL" : "BUY";

  const timestamp = Date.now();
  const query = `symbol=${SYMBOL}&side=${side}&type=MARKET&quantity=${activeTrade.quantity}&timestamp=${timestamp}`;

  const signature = crypto
    .createHmac("sha256", SECRET_KEY)
    .update(query)
    .digest("hex");

  await axios.post(
    `https://api.binance.com/api/v3/order?${query}&signature=${signature}`,
    null,
    { headers: { "X-MBX-APIKEY": API_KEY } }
  );

  const result =
    (activeTrade.side === "BUY" && price > activeTrade.entry) ||
    (activeTrade.side === "SELL" && price < activeTrade.entry)
      ? "WIN"
      : "LOSS";

  tradeHistory.push({ result });

  if (result === "LOSS") lossCount++;

  activeTrade = null;
}

// ===== MAIN LOOP =====
setInterval(async () => {
  if (!botRunning) return;

  if (tradesToday >= 3 || lossCount >= 2) {
    console.log("Risk limits reached");
    return;
  }

  try {
    const closes = await getMarketData();
    const result = analyze(closes);

    console.log(result);

    await manageTrade(result.price);

    if (!activeTrade && result.signal !== "NO TRADE") {
      await placeOrder(result.signal, result.price);
    }

  } catch (err) {
    console.error(err.message);
  }

}, 15000);

// ===== API CONTROL =====
app.post("/start", (req, res) => {
  botRunning = true;
  res.send("Bot started");
});

app.post("/stop", (req, res) => {
  botRunning = false;
  res.send("Bot stopped");
});

app.get("/status", (req, res) => {
  res.json({ botRunning, tradesToday, lossCount });
});

app.listen(3000, () => console.log("Bot running..."));https://github.com/mndebelecollen074-netizen/Trading-bot-.git
