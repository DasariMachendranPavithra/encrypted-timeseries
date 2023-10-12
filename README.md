const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const crypto = require('crypto');
const fs = require('fs');
const { TimeSeriesDatabase } = require('timeseries-database-library');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

const data = JSON.parse(fs.readFileSync('data.json'));

const generateRandomMessage = () => {
  const name = data.names[Math.floor(Math.random() * data.names.length)];
  const origin = data.locations[Math.floor(Math.random() * data.locations.length)];
  const destination = data.locations[Math.floor(Math.random() * data.locations.length)];
  const secretKey = crypto
    .createHash('sha256')
    .update(name + origin + destination)
    .digest('hex');

  return {
    name,
    origin,
    destination,
    secretKey,
  };
};

const emitRandomMessages = () => {
  const messageCount = Math.floor(Math.random() * 451) + 49;
  for (let i = 0; i < messageCount; i++) {
    const message = generateRandomMessage();
    io.emit('encrypted-message', message);
  }
};

// Replace this with your Time Series Database logic
const saveToTimeSeriesDB = (message) => {
  // Implement saving the data to your time series database.
};

io.on('connection', (socket) => {
  console.log('Client connected');
  emitRandomMessages();

  socket.on('disconnect', () => {
    console.log('Client disconnected');
  });

  socket.on('decrypted-message', (message) => {
    // Replace this with your decryption logic
    // After decryption, you can save the message to the time series database.
    saveToTimeSeriesDB(message);
  });
});

server.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
