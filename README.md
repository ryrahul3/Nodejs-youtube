Here’s how you can design the API using Node.js with MySQL for the YouTube trending videos assignment, along with a free MySQL hosting platform recommendation.

---

### **1. API Design Overview**
We'll create three main endpoints:
1. **GET `/api/fetch-trending`**: Scrapes YouTube trending videos and saves them to the database.
2. **GET `/api/videos`**: Fetches the list of videos from the database.
3. **GET `/api/videos/:id`**: Fetches detailed information about a specific video from the database.

---

### **2. MySQL Setup**
You can use a free MySQL hosting platform such as:
- **[PlanetScale](https://planetscale.com/)**: A scalable, free MySQL database provider with an easy-to-use interface.
- **[FreeMySQLHosting.net](https://www.freemysqlhosting.net/)**: Offers free MySQL hosting with limitations.

Once you register on one of these platforms:
1. Create a new database.
2. Note down the credentials: host, username, password, and database name.

---

### **3. Node.js Project Setup**
1. **Initialize Project**:
   ```bash
   mkdir youtube-trending-api
   cd youtube-trending-api
   npm init -y
   npm install express mysql axios cheerio dotenv body-parser
   ```

2. **Directory Structure**:
   ```
   youtube-trending-api/
   ├── server.js
   ├── db.js
   ├── routes/
   │   └── videos.js
   ├── models/
   │   └── VideoModel.js
   └── .env
   ```

---

### **4. Code Implementation**

#### **`server.js`**
Main entry point for the server.
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const videoRoutes = require('./routes/videos');

const app = express();
app.use(bodyParser.json());
app.use('/api', videoRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

#### **`db.js`**
Database connection.
```javascript
const mysql = require('mysql');
require('dotenv').config();

const connection = mysql.createConnection({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME
});

connection.connect(err => {
    if (err) throw err;
    console.log('Database connected.');
});

module.exports = connection;
```

#### **`routes/videos.js`**
API routes for handling video-related operations.
```javascript
const express = require('express');
const axios = require('axios');
const cheerio = require('cheerio');
const db = require('../db');
const { saveVideo, getVideos, getVideoById } = require('../models/VideoModel');

const router = express.Router();

// Fetch trending videos
router.get('/fetch-trending', async (req, res) => {
    try {
        const response = await axios.get('https://www.youtube.com/feed/trending');
        const $ = cheerio.load(response.data);

        // Extract video data (mockup example, adjust according to YouTube's HTML structure)
        const videos = [];
        $('.video-title').each((i, element) => {
            const title = $(element).text();
            const url = $(element).attr('href');
            videos.push({ title, url });
        });

        for (const video of videos) {
            await saveVideo(video);
        }

        res.status(200).send('Trending videos fetched and saved.');
    } catch (err) {
        console.error(err);
        res.status(500).send('Failed to fetch trending videos.');
    }
});

// Get all videos
router.get('/videos', (req, res) => {
    getVideos((err, results) => {
        if (err) return res.status(500).send('Failed to fetch videos.');
        res.json(results);
    });
});

// Get video details
router.get('/videos/:id', (req, res) => {
    const { id } = req.params;
    getVideoById(id, (err, result) => {
        if (err) return res.status(500).send('Failed to fetch video details.');
        res.json(result);
    });
});

module.exports = router;
```

#### **`models/VideoModel.js`**
Video database operations.
```javascript
const db = require('../db');

const saveVideo = (video, callback) => {
    const { title, url } = video;
    const query = `
        INSERT INTO videos (title, url)
        VALUES (?, ?)
        ON DUPLICATE KEY UPDATE
        title = VALUES(title), url = VALUES(url);
    `;
    db.query(query, [title, url], callback);
};

const getVideos = callback => {
    db.query('SELECT * FROM videos', callback);
};

const getVideoById = (id, callback) => {
    db.query('SELECT * FROM videos WHERE id = ?', [id], callback);
};

module.exports = { saveVideo, getVideos, getVideoById };
```

#### **`.env`**
Store sensitive credentials.
```plaintext
DB_HOST=your-db-host
DB_USER=your-db-username
DB_PASSWORD=your-db-password
DB_NAME=your-db-name
```

---

### **5. Database Schema**
Run the following SQL script to create the `videos` table:
```sql
CREATE TABLE videos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    url VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

### **6. Deploy Free MySQL Server**
#### **Using PlanetScale**:
1. Sign up at [PlanetScale](https://planetscale.com/).
2. Create a new database.
3. Note the connection details and update your `.env` file with them.

---

### **7. Hosting Platforms**
You can deploy this app to a free platform like:
1. **Render** (with MySQL support).
2. **Railway** (simple setup and supports MySQL integration).
3. **Heroku** (add a MySQL add-on).

Would you like detailed deployment instructions for one of these platforms?
