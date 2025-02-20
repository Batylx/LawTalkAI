from flask import Flask, request, jsonify
import openai

app = Flask(__name__)
openai.api_key = "your-openai-api-key"  # Wstaw swój klucz OpenAI

@app.route("/chat", methods=["GET"])
def chat_with_ai():
    query = request.args.get('query')  # Pytanie do AI
    if query:
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=query,
            max_tokens=150
        )
        return jsonify(response.choices[0].text.strip())
    return jsonify({"message": "Please provide a query."})

if __name__ == "__main__":
    app.run(debug=True)
