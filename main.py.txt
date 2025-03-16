from flask import Flask, request, jsonify
import openai
import os
from dotenv import load_dotenv

app = Flask(__name__)

# Загружаем API ключ из .env файла
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

PROMPT = """
Ты — эксперт по анализу контента. Клиент предоставил текст или описание своего бизнеса. 
Твоя задача — проанализировать его и дать рекомендации по улучшению и адаптации под его целевую аудиторию.
Определи сильные и слабые стороны, стиль общения и предложения по улучшению.
Текст или описание: 
"""

@app.route('/analyze', methods=['POST'])
def analyze():
    data = request.json
    user_input = data.get('text')

    if not user_input:
        return jsonify({"error": "No text provided"}), 400

    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": "Ты — эксперт по анализу контента."},
                {"role": "user", "content": PROMPT + user_input}
            ],
            max_tokens=1000,
            temperature=0.7,
        )
        result = response['choices'][0]['message']['content']
        return jsonify({"result": result})

    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
