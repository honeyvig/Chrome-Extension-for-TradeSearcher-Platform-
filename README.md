# Chrome-Extension-for-TradeSearcher-Platform
create a Chrome Extension that connect with our platform TradeSearcher and is meant to be used on TradingView. The chrome extension will allow users to:

- Log in to their account
- Add a section into TradingView Pro Chart at the bottom.
- Auto display relevant trading strategies by fetching the TradeSearcher API automatically when  navigating a particular symbol/ticker on TradingView
- Basic sort/filter
- Ability to add a strategy to favourites
- Scanner: Allow user to test any strategy on TradingView on X number of symbols he select (from an existing/new list of symbols to backtests in bulk)
- A few extension settings: Auto-Add Strategy to chart (Yes/No)


Requirements:
- Ability to debug/isolate TradingView app API calls to perform action with the extension (mass scanning, add to favorite, add/remove strategy to chart)
- Excellent code abilities
- Experience in creating Chrome Extensions that modify the DOM, log in to external tools and save data locally
- Expérience in front-end development
-------------------
To create the Chrome Extension you’ve described, which connects to TradeSearcher and interacts with TradingView, we’ll break it down into key components. This extension will include features such as logging in, interacting with the TradingView charts, fetching strategies from the TradeSearcher API, and managing a strategy list. We'll use JavaScript, HTML, CSS, and Chrome Extension APIs for the implementation. Here's how you can approach this task step by step:
1. Setting Up the Chrome Extension Structure

First, create the following file structure for your extension:

trade-searcher-extension/
│
├── manifest.json
├── popup.html
├── popup.js
├── content.js
├── background.js
├── styles.css
├── icons/
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
└── api/
    └── tradeSearcherAPI.js

2. Creating the manifest.json

This is the main configuration file for your Chrome Extension. It will define permissions, background scripts, and content scripts that modify the TradingView page.

{
  "manifest_version": 2,
  "name": "TradeSearcher for TradingView",
  "description": "Enhance TradingView with TradeSearcher strategies.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "https://api.tradesearcher.com/*",
    "https://www.tradingview.com/*"
  ],
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  },
  "content_scripts": [
    {
      "matches": ["https://www.tradingview.com/*"],
      "js": ["content.js"]
    }
  ],
  "browser_action": {
    "default_popup": "popup.html",
    "default_icon": "icons/icon48.png"
  },
  "options_page": "options.html",
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}

3. Creating the Popup (popup.html and popup.js)

This will be the UI for the extension where users can log in and see strategy-related data.

popup.html:

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>TradeSearcher Extension</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h2>TradeSearcher</h2>
  <button id="loginBtn">Login to TradeSearcher</button>
  <div id="strategyList">
    <h3>Available Strategies</h3>
    <ul id="strategies"></ul>
  </div>
  <div id="settings">
    <label for="autoAddStrategy">Auto-Add Strategy to Chart:</label>
    <input type="checkbox" id="autoAddStrategy">
  </div>
  <script src="popup.js"></script>
</body>
</html>

popup.js:

document.getElementById('loginBtn').addEventListener('click', login);
document.getElementById('autoAddStrategy').addEventListener('change', toggleAutoAdd);

function login() {
  // Handle user login (you could show a login modal or request API credentials)
  chrome.storage.local.set({ loggedIn: true });
  fetchStrategies();
}

function fetchStrategies() {
  // Call the TradeSearcher API to fetch relevant strategies based on the current symbol/ticker
  chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
    const url = tabs[0].url;
    const symbol = extractSymbolFromUrl(url);  // Parse the symbol/ticker from TradingView URL
    fetch(`https://api.tradesearcher.com/strategies?symbol=${symbol}`)
      .then(response => response.json())
      .then(data => displayStrategies(data));
  });
}

function displayStrategies(strategies) {
  const strategiesList = document.getElementById('strategies');
  strategiesList.innerHTML = '';
  strategies.forEach(strategy => {
    const li = document.createElement('li');
    li.innerHTML = `${strategy.name} <button onclick="addToFavorites('${strategy.id}')">Add to Favorites</button>`;
    strategiesList.appendChild(li);
  });
}

function addToFavorites(strategyId) {
  chrome.storage.local.get('favorites', function(data) {
    const favorites = data.favorites || [];
    if (!favorites.includes(strategyId)) {
      favorites.push(strategyId);
      chrome.storage.local.set({ favorites });
    }
  });
}

function toggleAutoAdd(event) {
  chrome.storage.local.set({ autoAddStrategy: event.target.checked });
}

function extractSymbolFromUrl(url) {
  const match = url.match(/symbol=([A-Za-z0-9]+)/);
  return match ? match[1] : '';
}

4. Connecting with the TradeSearcher API (api/tradeSearcherAPI.js)

In this file, you will connect to TradeSearcher API for fetching relevant strategies.

// api/tradeSearcherAPI.js

async function fetchStrategies(symbol) {
  const response = await fetch(`https://api.tradesearcher.com/strategies?symbol=${symbol}`);
  const data = await response.json();
  return data;
}

5. Content Script (content.js)

This script will interact with the TradingView chart and embed the TradeSearcher data below it.

content.js:

chrome.storage.local.get(['loggedIn', 'autoAddStrategy'], function(data) {
  if (data.loggedIn) {
    addTradeSearcherSection();
  }
});

function addTradeSearcherSection() {
  const chartContainer = document.querySelector('.tv-chart-container');
  if (chartContainer) {
    const tradeSearcherDiv = document.createElement('div');
    tradeSearcherDiv.id = 'tradeSearcherSection';
    tradeSearcherDiv.style.cssText = 'height: 200px; background: #f4f4f4; margin-top: 10px; padding: 10px;';
    
    const strategiesContainer = document.createElement('div');
    strategiesContainer.id = 'strategiesContainer';
    tradeSearcherDiv.appendChild(strategiesContainer);

    chartContainer.appendChild(tradeSearcherDiv);

    fetchStrategiesAndUpdateSection();
  }
}

function fetchStrategiesAndUpdateSection() {
  chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
    const url = tabs[0].url;
    const symbol = extractSymbolFromUrl(url);

    fetch(`https://api.tradesearcher.com/strategies?symbol=${symbol}`)
      .then(response => response.json())
      .then(data => {
        const strategiesContainer = document.getElementById('strategiesContainer');
        strategiesContainer.innerHTML = '';
        data.forEach(strategy => {
          const strategyDiv = document.createElement('div');
          strategyDiv.innerText = strategy.name;
          strategiesContainer.appendChild(strategyDiv);
        });
      });
  });
}

function extractSymbolFromUrl(url) {
  const match = url.match(/symbol=([A-Za-z0-9]+)/);
  return match ? match[1] : '';
}

6. Background Script (background.js)

This handles extension-wide events like storing settings and managing API calls.

chrome.runtime.onInstalled.addListener(function () {
  chrome.storage.local.set({ loggedIn: false, favorites: [] });
});

chrome.runtime.onMessage.addListener(function (request, sender, sendResponse) {
  if (request.action === 'fetchStrategies') {
    fetchStrategiesFromAPI(request.symbol).then(strategies => {
      sendResponse({ strategies });
    });
  }
  return true;  // Keep the message channel open for async response
});

async function fetchStrategiesFromAPI(symbol) {
  const response = await fetch(`https://api.tradesearcher.com/strategies?symbol=${symbol}`);
  const data = await response.json();
  return data;
}

7. Styling the Extension (styles.css)

You can add some basic styles for the popup and elements in your extension.

/* styles.css */
#strategies {
  margin-top: 10px;
}

button {
  background-color: #4CAF50;
  color: white;
  border: none;
  padding: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

8. Conclusion and Testing

Now that the basic structure is set up, the next steps would be:

    Testing the extension on TradingView and making sure that it fetches the correct strategies from the TradeSearcher API.
    Debugging any issues related to DOM manipulation (adding/removing strategies from charts).
    Setting up user authentication flow and securely storing the login credentials.
    Polishing the UI/UX and ensuring that the extension behaves correctly when navigating between different symbols/tickers.

Once you’re confident in its functionality, you can publish the extension to the Chrome Web Store following the Chrome Web Store publishing process.
