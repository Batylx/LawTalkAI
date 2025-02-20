from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename
import os
from services.document_processor import process_document
from services.google_docs_integration import integrate_google_docs

app = Flask(__name__)

# Konfiguracja folderu dla uploadu plików
app.config['UPLOAD_FOLDER'] = './uploads'
app.config['ALLOWED_EXTENSIONS'] = {'pdf', 'docx'}

# Funkcja sprawdzająca, czy plik jest dozwolony
def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in app.config['ALLOWED_EXTENSIONS']

# Endpoint do wgrywania umowy
@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({"error": "No file part"}), 400
    file = request.files['file']
    if file.filename == '':
        return jsonify({"error": "No selected file"}), 400
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(filepath)
        
        # Przetwarzanie dokumentu
        results = process_document(filepath)
        return jsonify(results), 200
    return jsonify({"error": "File type not allowed"}), 400

if __name__ == '__main__':
    app.run(debug=True)

    import pytesseract
from pdfminer.high_level import extract_text
import docx

# Funkcja do przetwarzania dokumentu
def process_document(file_path):
    # Sprawdzanie rozszerzenia pliku
    if file_path.endswith('.pdf'):
        text = extract_text(file_path)
    elif file_path.endswith('.docx'):
        doc = docx.Document(file_path)
        text = "\n".join([para.text for para in doc.paragraphs])
    else:
        text = pytesseract.image_to_string(file_path)  # OCR dla skanów

    # Analiza kluczowych klauzul (tutaj przykładowo)
    clauses = analyze_clauses(text)

    return clauses

# Funkcja do analizy klauzul w tekście
def analyze_clauses(text):
    clauses = {
        "penalties": "analiza kar umownych",
        "termination_period": "analiza okresów wypowiedzenia",
        "hidden_fees": "szukanie ukrytych opłat",
    }

    # Implementacja analizy (to tylko przykładowa funkcjonalność)
    results = {key: f"Znaleziono klauzulę: {value}" for key, value in clauses.items()}
    
    return results

    from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

# Funkcja do integracji z Google Docs
def integrate_google_docs(document_id):
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)

    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)

        with open('token.json', 'w') as token:
            token.write(creds.to_json())

    service = build('docs', 'v1', credentials=creds)
    document = service.documents().get(documentId=document_id).execute()

    return document

    import React, { useState } from 'react';

const UploadForm = () => {
    const [file, setFile] = useState(null);
    const [report, setReport] = useState(null);

    const handleFileChange = (e) => {
        setFile(e.target.files[0]);
    }

    const handleUpload = async () => {
        const formData = new FormData();
        formData.append('file', file);

        const response = await fetch('http://localhost:5000/upload', {
            method: 'POST',
            body: formData
        });

        const result = await response.json();
        setReport(result);
    }

    return (
        <div>
            <h1>Upload Umowy</h1>
            <input type="file" onChange={handleFileChange} />
            <button onClick={handleUpload}>Prześlij</button>

            {report && (
                <div>
                    <h2>Wyniki analizy:</h2>
                    <pre>{JSON.stringify(report, null, 2)}</pre>
                </div>
            )}
        </div>
    );
}

export default UploadForm;
