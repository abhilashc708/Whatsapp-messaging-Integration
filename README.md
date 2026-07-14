# Whatsapp-messaging-Integration(**Baileys**) 
Whatsapp messaging integration on application

Architechture
-------------

Temple Billing (Spring Boot)
        |
        | REST API
        |
        v
WhatsApp Service (Node.js + Baileys)
        |
        v
WhatsApp

The service will expose APIs like:
POST /send-message
POST /send-document
GET /status


**STEP 1** - Create a new project

cd ~/Documents/Works
mkdir whatsapp-service
cd whatsapp-service

Initialize Node:

**npm init -y**


**STEP 2** - Install packages

**npm install express cors multer dotenv**

**npm install @whiskeysockets/baileys**

**npm install qrcode-terminal**

**npm install pino**



At the end your package.json should contain packages similar to:

{
  "dependencies": {
    "@whiskeysockets/baileys": "...",
    "cors": "...",
    "dotenv": "...",
    "express": "...",
    "multer": "...",
    "pino": "...",
    "qrcode-terminal": "..."
  }
}

**STEP 3** - Create folder structure

whatsapp-service
│
├── auth
├── uploads
├── src
│   ├── app.js
│   ├── routes
│   ├── services
│   ├── controllers
│   └── config
│
├── package.json
└── .env

create directories

mkdir auth
mkdir uploads
mkdir src
mkdir src/routes
mkdir src/services
mkdir src/controllers
mkdir src/config


**STEP 4** - Create .env

.env (touch .env  and nano .env put PORT=3000 and save)


**STEP 5** - Create app.js

src/app.js (touch src/app.js)


**STEP 6** - Verify installation

npm list @whiskeysockets/baileys  (@whiskeysockets/baileys@7.x.x)

**STEP 7** - Verify Node version

node -v (v22.13.1)


Expected project structure

whatsapp-service
│
├── auth
├── uploads
├── src
│   ├── app.js
│   ├── config
│   ├── controllers
│   ├── routes
│   └── services
│
├── .env
├── package.json
└── package-lock.json


**Step 8** - Connect Baileys to WhatsApp

**sub 1**:
src/services/whatsapp.service.js

copy below code

const {
    default: makeWASocket,
    useMultiFileAuthState,
    DisconnectReason,
    fetchLatestBaileysVersion
} = require("@whiskeysockets/baileys");

const qrcode = require("qrcode-terminal");

let sock;

async function connectWhatsapp() {

    const { state, saveCreds } =
        await useMultiFileAuthState("./auth");

    const { version } =
        await fetchLatestBaileysVersion();

    sock = makeWASocket({
        version,
        auth: state,
        printQRInTerminal: false
    });

    sock.ev.on("creds.update", saveCreds);

    sock.ev.on("connection.update", ({ connection, qr, lastDisconnect }) => {

        if (qr) {

            console.log("");
            console.log("==================================");
            console.log("Scan this QR using Temple WhatsApp");
            console.log("==================================");

            qrcode.generate(qr, {
                small: true
            });
        }

        if (connection === "open") {

            console.log("");
            console.log("==================================");
            console.log("WhatsApp Ready");
            console.log("Connected Successfully");
            console.log("==================================");
        }

        if (connection === "close") {

            const shouldReconnect =
                lastDisconnect?.error?.output?.statusCode !==
                DisconnectReason.loggedOut;

            console.log("Connection Closed");

            if (shouldReconnect) {
                connectWhatsapp();
            }
        }

    });

}

function getSocket() {
    return sock;
}

module.exports = {
    connectWhatsapp,
    getSocket,
    sendTextMessage
};
async function sendTextMessage(mobile, message) {

    if (!sock) {
        throw new Error("WhatsApp is not connected.");
    }

    const jid = mobile + "@s.whatsapp.net";

    await sock.sendMessage(jid, {
        text: message
    });

}

**sub 2**:

src/app.js

copy below code


require("dotenv").config();

const express = require("express");
const cors = require("cors");

const { connectWhatsapp } = require("./services/whatsapp.service");
const { getSocket } = require("./services/whatsapp.service");

const whatsappController =
    require("./controllers/whatsapp.controller");

const app = express();

app.use(cors());
app.use(express.json());
app.use("/api/whatsapp", whatsappController);

// Start WhatsApp connection
connectWhatsapp();

// Health Check API
app.get("/", (req, res) => {
    res.send("Temple WhatsApp Service Running");
});

// Status API
app.get("/status", (req, res) => {

    const sock = getSocket();

    if (!sock) {
        return res.json({
            success: false,
            connected: false,
            message: "WhatsApp not initialized"
        });
    }

    return res.json({
        success: true,
        connected: !!sock.user,
        user: sock.user || null
    });

});

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
    console.log("==================================");
    console.log(`Temple WhatsApp Service Started`);
    console.log(`http://localhost:${PORT}`);
    console.log("==================================");
});


run node src/app.js or npm start


Expected Output

==================================
Temple WhatsApp Service Started
http://localhost:3000
==================================

==================================
Scan this QR using Temple WhatsApp
==================================

█████████████████████
██ ▄▄▄▄▄ █ ▄█ ...
...



scan qr code, once done

==================================
WhatsApp Ready
Connected Successfully
==================================


Your project is now ready.
You have completed:
✅ Express Server
✅ Baileys installation
✅ QR Login
✅ Session Persistence
✅ Auto Reconnect
✅ WhatsApp Ready



**Step 9**

**9.1**

Now we'll expose REST APIs from this service.
We'll build:

POST /api/whatsapp/send-text

request
{
    "mobile":"919544350091",
    "message":"Thank you for your donation."
}


response

{
    "success": true,
    "message": "Message Sent Successfully"
}



**Step 9.2** - Create Controller

src/controllers/whatsapp.controller.js

const express = require("express");

const router = express.Router();

const {
    sendTextMessage
} = require("../services/whatsapp.service");

router.post("/send-text", async (req, res) => {

    try {

        const { mobile, message } = req.body;

        if (!mobile || !message) {

            return res.status(400).json({
                success: false,
                message: "Mobile and message are required"
            });

        }

        await sendTextMessage(mobile, message);

        res.json({
            success: true,
            message: "Message sent successfully"
        });

    } catch (err) {

        console.error(err);

        res.status(500).json({
            success: false,
            message: err.message
        });

    }

});

module.exports = router;


**Step 9.3** - Add a send method to whatsapp.service.js (already the code added above)

async function sendTextMessage(mobile, message) {

    if (!sock) {
        throw new Error("WhatsApp is not connected.");
    }

    const jid = mobile + "@s.whatsapp.net";

    await sock.sendMessage(jid, {
        text: message
    });

}




module.exports = {
    connectWhatsapp,
    getSocket,
    sendTextMessage
};



**Step 9.4**- Register the Route (already updated above code)

src/app.js

const whatsappController =
    require("./controllers/whatsapp.controller");

app.use("/api/whatsapp", whatsappController);


**Step 9.5** - Restart the Server

ctrl + c then   node src/app.js

wait until see

WhatsApp Ready
Connected Successfully

**Step 9.6** - Test with Postman
POST

http://localhost:3000/api/whatsapp/send-text

Content-Type: application/json

body json

{
    "mobile":"919544350091",
    "message":"🙏 Thank you for your donation to Mankurussi Bhagavathi Temple."
}

response

{
    "success": true,
    "message": "Message sent successfully"
}




