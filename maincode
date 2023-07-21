const express = require('express');
const mongoose = require('mongoose');
const { valid, parse } = require('url');

const app = express();

mongoose.connect('mongodb://0.0.0.0:27017/url_shortener', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const urlSchema = new mongoose.Schema({
  destinationUrl: {
    type: String,
    required: true,
    validate: {
      validator: (value) => {
        const urlObj = parse(value);
        return urlObj && urlObj.hostname;
      },
      message: 'Invalid URL format',
    },
  },
  shortUrl: String,
  expiryDate: Date,
});

const Url = mongoose.model('Url', urlSchema);

app.use(express.json());

app.post('/shorten', async (req, res) => {
  try {
    const { destinationUrl } = req.body;

    const shortCode = generateShortCode();

    const url = new Url({
      destinationUrl,
      shortUrl: shortCode,
    });

    await url.save();

    res.json({ shortUrl: shortCode });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.put('/update', async (req, res) => {
  try {
    const { shortUrl, destinationUrl } = req.body;

    const url = await Url.findOne({ shortUrl });

    if (!url) {
      return res.status(404).json({ error: 'URL not found' });
    }

    url.destinationUrl = destinationUrl;
    await url.save();

    res.json({ success: true });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.get('/:shortCode', async (req, res) => {
  try {
    const { shortCode } = req.params;

    const url = await Url.findOne({ shortUrl: shortCode });

    if (!url) {
      return res.status(404).json({ error: 'URL not found' });
    }

    res.json({ destinationUrl: url.destinationUrl });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.put('/expiry', async (req, res) => {
  try {
    const { shortUrl, daysToAdd } = req.body;

    const url = await Url.findOne({ shortUrl });

    if (!url) {
      return res.status(404).json({ error: 'URL not found' });
    }

    url.expiryDate.setDate(url.expiryDate.getDate() + daysToAdd);
    await url.save();

    res.json({ success: true });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

function generateShortCode() {
  const characters = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let shortCode = '';

  for (let i = 0; i < 8; i++) {
    const randomIndex = Math.floor(Math.random() * characters.length);
    shortCode += characters[randomIndex];
  }

  return shortCode;
}

app.listen(3000, () => {
  console.log('Server started on port 3000');
});
