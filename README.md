const express = require("express");
const multer = require("multer");
const cors = require("cors");
const path = require("path");
const fs = require("fs");

const app = express();
app.use(cors());
app.use(express.json());

const UPLOADS_DIR = path.join(__dirname, "uploads");
if (!fs.existsSync(UPLOADS_DIR)) fs.mkdirSync(UPLOADS_DIR);

const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, UPLOADS_DIR),
  filename: (req, file, cb) => cb(null, Date.now() + path.extname(file.originalname))
});
const upload = multer({ storage });

let videos = [];

app.post("/upload", upload.single("video"), (req, res) => {
  const { title, uploader } = req.body;
  if (!req.file || !title || !uploader) return res.status(400).send("Missing video, title or uploader");

  const video = { id: videos.length + 1, title, filename: req.file.filename, likes: 0, dislikes: 0, comments: [], uploader };
  videos.push(video);
  res.send({ message: "Upload OK", video });
});

app.get("/videos", (req, res) => res.json(videos));

app.post("/like", (req, res) => {
  const { id } = req.body;
  const vid = videos.find(v => v.id === id);
  if (!vid) return res.status(404).send("Video not found");
  vid.likes++;
  res.send({ likes: vid.likes });
});

app.post("/dislike", (req, res) => {
  const { id } = req.body;
  const vid = videos.find(v => v.id === id);
  if (!vid) return res.status(404).send("Video not found");
  vid.dislikes++;
  res.send({ dislikes: vid.dislikes });
});

app.post("/comment", (req, res) => {
  const { id, user, text } = req.body;
  const vid = videos.find(v => v.id === id);
  if (!vid) return res.status(404).send("Video not found");
  if (!user || !text) return res.status(400).send("Missing user or text");
  vid.comments.push({ user, text });
  res.send({ comments: vid.comments });
});

app.use("/uploads", express.static(UPLOADS_DIR));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`VHub backend running on port ${PORT}`));
