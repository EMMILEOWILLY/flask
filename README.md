# flask

from flask import Flask, request, jsonify
import openai
import sqlite3

app = Flask(__name__)

# OpenAI API Key (Replace with your actual key)
openai.api_key = "your_openai_api_key"

# Function to interact with OpenAI API
def get_ai_response(question):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "system", "content": "You are an AI tutor."},
                  {"role": "user", "content": question}]
    )
    return response["choices"][0]["message"]["content"]

# Route to handle chat requests
@app.route('/chat', methods=['POST'])
def chat():
    data = request.json
    user_question = data.get("question")
    
    if not user_question:
        return jsonify({"error": "No question provided"}), 400
    
    ai_response = get_ai_response(user_question)

    # Store chat history in SQLite
    conn = sqlite3.connect("chat_history.db")
    cursor = conn.cursor()
    cursor.execute("INSERT INTO chats (user_question, ai_response) VALUES (?, ?)", (user_question, ai_response))
    conn.commit()
    conn.close()

    return jsonify({"response": ai_response})

# Initialize database
def init_db():
    conn = sqlite3.connect("chat_history.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS chats (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_question TEXT,
            ai_response TEXT
        )
    """)
    conn.commit()
    conn.close()

if __name__ == '__main__':
    init_db()
    app.run(debug=True)

