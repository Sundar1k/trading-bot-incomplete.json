# trading-bot-incomplete.json
WIP n8n workflow for crypto trading bot. Includes API placeholders and basic logic. Not yet complete.
{
  "name": "My workflow",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "/webhook",
        "options": {}
      },
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [
        -320,
        0
      ],
      "id": "a426ac14-077d-4eb4-b656-870c7dd71112",
      "name": "Webhook",
      "webhookId": "9b396d31-1c9d-4a4d-8918-a5331af10323"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "const symbols = $input.item.json.symbols || ['BTCUSDT', 'ETHUSDT'];\nreturn { json: { symbols } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        -120,
        0
      ],
      "id": "55ea4ceb-e6bc-4396-98eb-dbc249a2befd",
      "name": "Code"
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 3,
      "position": [
        100,
        0
      ],
      "id": "cd6eaa8f-2f36-45be-8bca-6275330694a9",
      "name": "Loop Over Items"
    },
    {
      "parameters": {},
      "type": "n8n-nodes-base.noOp",
      "name": "Replace Me",
      "typeVersion": 1,
      "position": [
        320,
        0
      ],
      "id": "e95d487e-fb8c-41fd-b521-0afac8ad9a1e"
    },
    {
      "parameters": {
        "url": "https://api.binance.com/api/v3/exchangeInfo",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        540,
        0
      ],
      "id": "5de3afba-2d39-4eda-a860-51655d779ae6",
      "name": "HTTP Request"
    },
    {
      "parameters": {
        "url": "https://api.binance.com/api/v3/klines",
        "sendQuery": true,
        "specifyQuery": "json",
        "jsonQuery": "=symbol: ={{$json.symbol}}\ninterval: 1h",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        760,
        0
      ],
      "id": "fe73d647-ea6d-4524-b62e-2bfdb28a48ae",
      "name": "HTTP Request1"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "const klines = $input.item.json.klines;\n\n// Extract close prices\nconst closes = klines.map(k => parseFloat(k[4]));\n\n// Calculate EMA (Exponential Moving Average)\nlet ema = 0;\nconst period = 20;\nlet multiplier = 2 / (period + 1);\n\nfor (let i = 0; i < closes.length; i++) {\n  if (i === 0) {\n    ema = closes[i];\n  } else {\n    ema = (closes[i] - ema) * multiplier + ema;\n  }\n}\n\n// Calculate RSI (Relative Strength Index)\nconst rsiPeriod = 14;\nlet gains = 0;\nlet losses = 0;\n\nfor (let i = 1; i < closes.length; i++) {\n  const change = closes[i] - closes[i - 1];\n  if (change > 0) {\n    gains += change;\n  } else {\n    losses += Math.abs(change);\n  }\n}\n\nconst avgGain = gains / rsiPeriod;\nconst avgLoss = losses / rsiPeriod;\nconst rs = avgGain / avgLoss;\nconst rsi = 100 - (100 / (1 + rs));\n\nconst price = closes[closes.length - 1];\nconst prevPrice = closes[closes.length - 2];\n\nreturn { json: { price, prev_price: prevPrice, ema, rsi } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        980,
        0
      ],
      "id": "80472d28-ba43-44c1-9fed-87a32c29663e",
      "name": "Code1"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "let signal = 'HOLD';\n\nconst price = $input.item.json.price;\nconst ema = $input.item.json.ema;\nconst rsi = $input.item.json.rsi;\nconst strategy = $input.item.json.strategy;\nconst strategyParams = $input.item.json.strategyParams || {};\n\nswitch (strategy) {\n  case 'day':\n    if (price > ema && rsi < strategyParams.rsi_upper_band) signal = 'BUY';\n    else if (price < ema && rsi > strategyParams.rsi_lower_band) signal = 'SELL';\n    break;\n\n  case 'swing':\n    if (price > ema && price > $input.item.json.prev_price) signal = 'BUY';\n    else if (price < ema && price < $input.item.json.prev_price) signal = 'SELL';\n    break;\n\n  case 'hodl':\n    if (price < ema) signal = 'BUY';\n    break;\n\n  default:\n    signal = 'HOLD';\n}\n\nreturn { json: { signal: signal.toUpperCase() } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1200,
        0
      ],
      "id": "eb240507-5a72-4097-bff1-8e351de9b8b3",
      "name": "Code2"
    },
    {
      "parameters": {
        "url": "https://api.binance.com/api/v3/account",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        1420,
        0
      ],
      "id": "3084f30a-6236-456b-be74-b1707ffc69ad",
      "name": "HTTP Request2"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "const quoteAsset = 'USDT';\nconst balances = $node['Get Binance Balance'].json.balances;\nconst quoteBalance = balances.find(b => b.asset === quoteAsset).free;\nconst riskPercentage = $input.item.json.riskPercentage || 0.01;\nconst amountToRisk = parseFloat(quoteBalance) * riskPercentage;\nconst price = $input.item.json.price;\n\n// Get symbol details from exchange info\nconst symbolDetails = $node['Get Exchange Info'].json.symbols.find(s => s.symbol === $input.item.json.symbol);\nconst minNotional = parseFloat(symbolDetails.filters.find(f => f.filterType === 'MIN_NOTIONAL').minNotional);\nconst stepSize = parseFloat(symbolDetails.orderBooks.find(o => o.symbol === $input.item.json.symbol).stepSize);\n\nlet quantity = amountToRisk / price;\nquantity = Math.round(quantity / stepSize) * stepSize;\n\n// Ensure minimum notional is met\nif (quantity * price < minNotional) {\n  quantity = minNotional / price;\n}\n\nreturn { json: { quantity } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        540,
        220
      ],
      "id": "fa349322-359e-474a-9be2-cf066feb67c8",
      "name": "Code3"
    },
    {
      "parameters": {
        "method": "POST",
        "url": " https://api.binance.com/api/v3/order",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"symbol\": \"={{$json.symbol}}\",\n  \"side\": \"={{$json.signal}}\",\n  \"type\": \"LIMIT\",\n  \"timeInForce\": \"GTC\",\n  \"quantity\": \"={{$json.quantity}}\",\n  \"price\": \"={{$json.price * 0.995}}\"\n}",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        760,
        220
      ],
      "id": "a7638d58-17be-4047-ae96-662e99df2ffc",
      "name": "HTTP Request3"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "const tradeApiResponse = $input.item.json;\n\nconst tradeData = {\n  orderId: tradeApiResponse.orderId,\n  symbol: tradeApiResponse.symbol,\n  status: tradeApiResponse.status,\n  side: tradeApiResponse.side,\n  price: tradeApiResponse.avgPrice,\n  quantity: tradeApiResponse.executedQty,\n  fees: tradeApiResponse.commission,\n  timestamp: new Date().toISOString()\n};\n\nreturn { json: { tradeData } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        980,
        220
      ],
      "id": "d568882f-f266-42bf-bb16-e778b1746900",
      "name": "Code4"
    },
    {
      "parameters": {
        "jsCode": "const experience = {\n  symbol: $input.item.json.symbol,\n  strategyUsed: $input.item.json.strategy,\n  signal: $input.item.json.signal,\n  price: $input.item.json.price,\n  ema: $input.item.json.ema,\n  rsi: $input.item.json.rsi,\n  actualFilledPrice: $input.item.json.tradeData.price,\n  quantityFilled: $input.item.json.tradeData.quantity,\n  feesPaid: $input.item.json.tradeData.fees,\n  timestamp: $input.item.json.tradeData.timestamp\n};\n\nreturn { json: { experience } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1200,
        220
      ],
      "id": "8e3a4fee-ca72-44b6-8cfb-5f7ee2eb101c",
      "name": "Code5"
    },
    {
      "parameters": {
        "operation": "write",
        "fileName": "D:\\\\N8N\\\\experiences.json",
        "options": {
          "append": "={\n  \"symbol\": \"={{$json.experience.symbol}}\",\n  \"signal\": \"={{$json.experience.signal}}\",\n  \"price\": \"={{$json.experience.price}}\",\n  \"actualFilledPrice\": \"={{$json.experience.actualFilledPrice}}\",\n  \"quantity\": \"={{$json.experience.quantityFilled}}\",\n  \"fees\": \"={{$json.experience.feesPaid}}\",\n  \"timestamp\": \"={{$json.experience.timestamp}}\"\n}"
        }
      },
      "type": "n8n-nodes-base.readWriteFile",
      "typeVersion": 1,
      "position": [
        1420,
        220
      ],
      "id": "00e2d20c-aa9c-4b43-828d-67aaf83ccd63",
      "name": "Read/Write Files from Disk"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "const experience = $input.item.json.experience;\nconst profit = (experience.actualFilledPrice - experience.price) * experience.quantity;\n\n// Simple learning logic: track profit/loss\nreturn { json: { updatedModel: true, profit } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        560,
        480
      ],
      "id": "ddc8fc1c-567f-4bea-81c6-a18e4a0445fa",
      "name": "Code6"
    },
    {
      "parameters": {
        "chatId": "Commandermonkeybot",
        "text": "N8N",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        780,
        480
      ],
      "id": "0d625168-a712-410a-83bc-5e30ac792bfa",
      "name": "Telegram",
      "webhookId": "c1316c91-d933-4f3c-bdd8-13afaecf0556",
      "credentials": {
        "telegramApi": {
          "id": "zmAVDGJHNgXZBxKv",
          "name": "Telegram account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 2
          },
          "conditions": [
            {
              "id": "9dc89cb7-9090-457c-af8d-2d3d6420efa9",
              "leftValue": "={{ $input.item.json.tradeData.status }}",
              "rightValue": "FILLED",
              "operator": {
                "type": "string",
                "operation": "equals",
                "name": "filter.operator.equals"
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [
        1000,
        480
      ],
      "id": "5b51005a-4de0-4274-85c0-9ec570a5284d",
      "name": "If",
      "executeOnce": false
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "console.log('Trade successful:', $input.item.json);\nreturn { json: { success: true } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1240,
        460
      ],
      "id": "8b84c2c1-2a52-4216-8369-8e8ebaf6a12b",
      "name": "Code7"
    },
    {
      "parameters": {
        "mode": "runOnceForEachItem",
        "jsCode": "console.error('Trade failed:', $input.item.json);\nreturn { json: { success: false } };"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1240,
        640
      ],
      "id": "6c2921ae-4668-40f4-bb7d-e5d87011813c",
      "name": "Code8"
    },
    {
      "parameters": {
        "mode": "raw",
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        1460,
        460
      ],
      "id": "8ff0238e-9774-4778-97d3-4057f919cb71",
      "name": "Edit Fields"
    }
  ],
  "pinData": {},
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "Code",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code": {
      "main": [
        [
          {
            "node": "Loop Over Items",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Loop Over Items": {
      "main": [
        [],
        [
          {
            "node": "Replace Me",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Replace Me": {
      "main": [
        [
          {
            "node": "Loop Over Items",
            "type": "main",
            "index": 0
          },
          {
            "node": "HTTP Request",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request": {
      "main": [
        [
          {
            "node": "HTTP Request1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request1": {
      "main": [
        [
          {
            "node": "Code1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code1": {
      "main": [
        [
          {
            "node": "Code2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code2": {
      "main": [
        [
          {
            "node": "HTTP Request2",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request2": {
      "main": [
        [
          {
            "node": "Code3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code3": {
      "main": [
        [
          {
            "node": "HTTP Request3",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "HTTP Request3": {
      "main": [
        [
          {
            "node": "Code4",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code4": {
      "main": [
        [
          {
            "node": "Code5",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code5": {
      "main": [
        [
          {
            "node": "Read/Write Files from Disk",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Read/Write Files from Disk": {
      "main": [
        [
          {
            "node": "Code6",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code6": {
      "main": [
        [
          {
            "node": "Telegram",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Telegram": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "Code7",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Code8",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Code7": {
      "main": [
        [
          {
            "node": "Edit Fields",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "9ed8e750-9425-452f-9b4c-018df35fb440",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "07e763ea7dab4c5f064d2eeb8991bcdb7a47e4ce8b9a83a4763a67109e3bff0e"
  },
  "id": "Mxor6cWHu5glbsmx",
  "tags": []
}
