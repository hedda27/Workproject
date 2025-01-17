# Workproject
from flask import Flask, request, jsonify
import openai

# Initialize Flask app
app = Flask(__name__)

# Set OpenAI API key
openai.api_key = "your_openai_api_key"

@app.route('/chat', methods=['POST'])
def chat():
    user_input = request.json.get("message", "")
    
    if not user_input:
        return jsonify({"error": "Message cannot be empty"}), 400

    try:
        # Call OpenAI API
        response = openai.Completion.create(
            engine="text-davinci-003",  # Choose the appropriate model
            prompt=user_input,
            max_tokens=150,
            temperature=0.7,
            top_p=1,
            frequency_penalty=0,
            presence_penalty=0.6,
            stop=["\n"]
        )
        
        bot_response = response.choices[0].text.strip()
        return jsonify({"response": bot_response})
    
    except Exception as e:
        return jsonify({"error": str(e)}), 500

# Run the Flask app
if __name__ == '__main__':
    app.run(debug=True)
    from flask import Flask, request, jsonify
import openai

app = Flask(__name__)

# Set OpenAI API Key
openai.api_key = "your_openai_api_key"

# In-memory storage for user sessions
user_sessions = {}

@app.route('/chat', methods=['POST'])
def chat():
    user_id = request.json.get("user_id")
    message = request.json.get("message")
    
    if not user_id or not message:
        return jsonify({"error": "User ID and message are required"}), 400
    
    # Initialize session if not present
    if user_id not in user_sessions:
        user_sessions[user_id] = []

    # Add user message to context
    user_sessions[user_id].append({"role": "user", "content": message})
    
    try:
        # Generate AI response with context
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=user_sessions[user_id],
            max_tokens=200,
            temperature=0.7
        )
        
        # Extract response and add it to the session context
        bot_response = response['choices'][0]['message']['content']
        user_sessions[user_id].append({"role": "assistant", "content": bot_response})

        return jsonify({"response": bot_response})
    
    except Exception as e:
        return jsonify({"error": str(e)}), 500

# Clear session for a user (optional endpoint)
@app.route('/clear_session', methods=['POST'])
def clear_session():
    user_id = request.json.get("user_id")
    if user_id in user_sessions:
        del user_sessions[user_id]
        return jsonify({"message": "Session cleared successfully"})
    return jsonify({"error": "User session not found"}), 404

if __name__ == '__main__':
    app.run(debug=True)
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Chatbot</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        #chat-container { max-width: 600px; margin: 0 auto; }
        #messages { border: 1px solid #ccc; padding: 10px; height: 400px; overflow-y: scroll; margin-bottom: 10px; }
        .message { margin: 5px 0; }
        .user { text-align: right; }
        .bot { text-align: left; color: blue; }
    </style>
</head>
<body>
    <div id="chat-container">
        <div id="messages"></div>
        <input type="text" id="user-input" placeholder="Type your message here" style="width: 80%;">
        <button onclick="sendMessage()">Send</button>
    </div>

    <script>
        const userId = Math.random().toString(36).substr(2, 9); // Generate a random user ID
        const messagesDiv = document.getElementById('messages');

        async function sendMessage() {
            const userInput = document.getElementById('user-input').value;
            if (!userInput) return;

            // Display user message
            appendMessage(userInput, 'user');
            document.getElementById('user-input').value = '';

            // Send message to the backend
            try {
                const response = await fetch('http://127.0.0.1:5000/chat', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ user_id: userId, message: userInput })
                });
                const data = await response.json();

                if (data.response) {
                    appendMessage(data.response, 'bot');
                } else {
                    appendMessage("Error: " + (data.error || "Unknown error"), 'bot');
                }
            } catch (error) {
                appendMessage("Error connecting to server.", 'bot');
            }
        }

        function appendMessage(text, sender) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${sender}`;
            messageDiv.textContent = text;
            messagesDiv.appendChild(messageDiv);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }
    </script>
</body>
</html>
let recognition;
if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    recognition = new SpeechRecognition();
    recognition.lang = 'en-US';
    recognition.continuous = false;

    recognition.onresult = (event) => {
        const transcript = event.results[0][0].transcript;
        document.getElementById('user-input').value = transcript;
        sendMessage();
    };

    recognition.onerror = (event) => {
        console.error('Speech recognition error:', event.error);
    };
}

async function speak(text) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = 'en-US';
    window.speechSynthesis.speak(utterance);
}

document.addEventListener('keydown', (e) => {
    if (e.key === 'v') {
        recognition.start();
    }
});

async function sendMessage() {
    const userInput = document.getElementById('user-input').value;
    if (!userInput) return;

    appendMessage(userInput, 'user');
    document.getElementById('user-input').value = '';

    try {
        const response = await fetch('http://127.0.0.1:5000/chat', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ user_id: userId, message: userInput })
        });
        const data = await response.json();

        if (data.response) {
            appendMessage(data.response, 'bot');
            speak(data.response); // Speak the response
        } else {
            appendMessage("Error: " + (data.error || "Unknown error"), 'bot');
        }
    } catch (error) {
        appendMessage("Error connecting to server.", 'bot');
    }
}
CREATE TABLE users (
    id TEXT PRIMARY KEY,
    name TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id TEXT,
    role TEXT,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users (id)
);

from flask import Flask, request, jsonify
import openai
import sqlite3
from datetime import datetime

app = Flask(__name__)
openai.api_key = "your_openai_api_key"

# Database connection
def get_db_connection():
    conn = sqlite3.connect('chatbot.db')
    conn.row_factory = sqlite3.Row
    return conn

@app.route('/chat', methods=['POST'])
def chat():
    user_id = request.json.get("user_id")
    message = request.json.get("message")
    
    if not user_id or not message:
        return jsonify({"error": "User ID and message are required"}), 400

    # Ensure the user exists
    conn = get_db_connection()
    user = conn.execute("SELECT * FROM users WHERE id = ?", (user_id,)).fetchone()
    if not user:
        conn.execute("INSERT INTO users (id) VALUES (?)", (user_id,))
        conn.commit()

    # Save user message to the database
    conn.execute("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)",
                 (user_id, "user", message))
    conn.commit()

    # Get the chat history
    messages = conn.execute("SELECT role, content FROM messages WHERE user_id = ? ORDER BY created_at",
                             (user_id,)).fetchall()
    conn.close()

    # Prepare chat history for the AI model
    conversation = [{"role": row["role"], "content": row["content"]} for row in messages]

    try:
        # Generate AI response
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=conversation,
            max_tokens=200,
            temperature=0.7
        )
        
        bot_response = response['choices'][0]['message']['content']

        # Save bot response to the database
        conn = get_db_connection()
        conn.execute("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)",
                     (user_id, "assistant", bot_response))
        conn.commit()
        conn.close()

        return jsonify({"response": bot_response})
    
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
import bcrypt  # Install using: pip install bcrypt

# Register a new user
@app.route('/register', methods=['POST'])
def register():
    data = request.json
    username = data.get('username')
    password = data.get('password')
    
    if not username or not password:
        return jsonify({"error": "Username and password are required"}), 400

    hashed_password = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
    try:
        conn = get_db_connection()
        conn.execute("INSERT INTO users (id, name) VALUES (?, ?)", (username, hashed_password))
        conn.commit()
        conn.close()
        return jsonify({"message": "User registered successfully"}), 201
    except sqlite3.IntegrityError:
        return jsonify({"error": "Username already exists"}), 400

# Authenticate a user
@app.route('/login', methods=['POST'])
def login():
    data = request.json
    username = data.get('username')
    password = data.get('password')
    
    if not username or not password:
        return jsonify({"error": "Username and password are required"}), 400

    conn = get_db_connection()
    user = conn.execute("SELECT * FROM users WHERE id = ?", (username,)).fetchone()
    conn.close()

    if user and bcrypt.checkpw(password.encode('utf-8'), user["name"].encode('utf-8')):
        return jsonify({"message": "Login successful", "user_id": username}), 200
    else:
        return jsonify({"error": "Invalid username or password"}), 401
# Total number of users
@app.route('/analytics/total_users', methods=['GET'])
def total_users():
    conn = get_db_connection()
    count = conn.execute("SELECT COUNT(*) AS user_count FROM users").fetchone()
    conn.close()
    return jsonify({"total_users": count["user_count"]})

# Top active users
@app.route('/analytics/top_users', methods=['GET'])
def top_users():
    conn = get_db_connection()
    results = conn.execute("""
        SELECT user_id, COUNT(*) AS message_count
        FROM messages
        GROUP BY user_id
        ORDER BY message_count DESC
        LIMIT 10
    """).fetchall()
    conn.close()
    return jsonify([{"user_id": row["user_id"], "message_count": row["message_count"]} for row in results])

# Most common topics (basic keyword tracking)
@app.route('/analytics/top_keywords', methods=['GET'])
def top_keywords():
    conn = get_db_connection()
    results = conn.execute("""
        SELECT content
        FROM messages
        WHERE role = 'user'
    """).fetchall()
    conn.close()

    # Analyze keywords
    from collections import Counter
    keywords = Counter()
    for row in results:
        keywords.update(row["content"].lower().split())

    return jsonify(keywords.most_common(10))        
 # Delete a user's conversation history
@app.route('/admin/clear_user_history', methods=['POST'])
def clear_user_history():
    user_id = request.json.get("user_id")
    if not user_id:
        return jsonify({"error": "User ID is required"}), 400

    conn = get_db_connection()
    conn.execute("DELETE FROM messages WHERE user_id = ?", (user_id,))
    conn.commit()
    conn.close()
    return jsonify({"message": f"Conversation history for {user_id} cleared successfully."})

# Ban a user
@app.route('/admin/ban_user', methods=['POST'])
def ban_user():
    user_id = request.json.get("user_id")
    if not user_id:
        return jsonify({"error": "User ID is required"}), 400

    conn = get_db_connection()
    conn.execute("UPDATE users SET banned = 1 WHERE id = ?", (user_id,))
    conn.commit()
    conn.close()
    return jsonify({"message": f"User {user_id} has been banned successfully."})

# List all users
@app.route('/admin/list_users', methods=['GET'])
def list_users():
    conn = get_db_connection()
    results = conn.execute("SELECT id, name, created_at FROM users").fetchall()
    conn.close()
    return jsonify([{"id": row["id"], "name": row["name"], "created_at": row["created_at"]} for row in results])
    
  @app.route('/export_chat', methods=['GET'])
def export_chat():
    user_id = request.args.get("user_id")
    if not user_id:
        return jsonify({"error": "User ID is required"}), 400

    conn = get_db_connection()
    messages = conn.execute("SELECT role, content, created_at FROM messages WHERE user_id = ? ORDER BY created_at",
                            (user_id,)).fetchall()
    conn.close()

    chat_export = [{"role": row["role"], "content": row["content"], "timestamp": row["created_at"]} for row in messages]

    # Save to a file (optional)
    with open(f"{user_id}_chat_history.json", "w") as file:
        import json
        json.dump(chat_export, file)

    return jsonify({"message": "Chat history exported successfully.", "data": chat_export})

from flask_socketio import SocketIO, emit

# Initialize Flask-SocketIO
socketio = SocketIO(app)

# Real-time chat
@socketio.on('message')
def handle_message(data):
    user_id = data.get("user_id")
    message = data.get("message")
    
    if not user_id or not message:
        emit("response", {"error": "User ID and message are required"})
        return

    # Save user message
    conn = get_db_connection()
    conn.execute("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)",
                 (user_id, "user", message))
    conn.commit()

    # Generate response
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": message}],
        max_tokens=200,
        temperature=0.7
    )
    bot_response = response['choices'][0]['message']['content']

    # Save bot response
    conn.execute("INSERT INTO messages (user_id, role, content) VALUES (?, ?, ?)",
                 (user_id, "assistant", bot_response))
    conn.commit()
    conn.close()

    emit("response", {"user_id": user_id, "response": bot_response})

@app.route('/summarize_context', methods=['POST'])
def summarize_context():
    user_id = request.json.get("user_id")
    if not user_id:
        return jsonify({"error": "User ID is required"}), 400

    conn = get_db_connection()
    messages = conn.execute("SELECT content FROM messages WHERE user_id = ? AND role = 'user'",
                            (user_id,)).fetchall()
    conn.close()

    conversation = " ".join([row["content"] for row in messages])

    # Summarize using OpenAI
    summary = openai.Completion.create(
        engine="text-davinci-003",
        prompt=f"Summarize this conversation: {conversation}",
        max_tokens=150
    )
    return jsonify({"summary": summary['choices'][0]['text'].strip()})

    
