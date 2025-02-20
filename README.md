from flask import Flask, request, jsonify
import openai

app = Flask(__LawTalkAI__)
openai.api_key = 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

@app.route("/chat", methods=["GET"])
def chat_with_ai():
    query = request.args.get('query')  
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
