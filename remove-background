import os
import sys
from flask import Flask, request, send_file, jsonify
import requests
from rembg import remove
from PIL import Image, UnidentifiedImageError
import io

# Suppress stdout and stderr
if os.name == 'posix':  
    sys.stdout = open(os.devnull, 'w')
    sys.stderr = open(os.devnull, 'w')
elif os.name == 'nt': 
    sys.stdout = open('nul', 'w')
    sys.stderr = open('nul', 'w')

# Initialize Flask app
app = Flask(__name__)

# Cache folder to store downloaded images
CACHE_FOLDER = 'cache'

# Create cache folder if it doesn't exist
if not os.path.exists(CACHE_FOLDER):
    os.makedirs(CACHE_FOLDER)

def download_image(image_url, filename):
    """Download image from the provided URL."""
    response = requests.get(image_url)
    if response.status_code == 200:
        with open(filename, 'wb') as f:
            f.write(response.content)
        return True
    return False

@app.route('/')
def index():
    """Background Remover API"""
    return 'Welcome to the Background Remover API! Use /rembg?url=<image_url> to remove background from an image.'

@app.route('/rembg', methods=['GET'])
def remove_background():
    """Remove background from the provided image."""
    image_url = request.args.get('url')
    if not image_url:
        return jsonify(error='Please provide a valid image URL.'), 400

    image_filename = os.path.join(CACHE_FOLDER, 'image.png')
    if not download_image(image_url, image_filename):
        return jsonify(error='Failed to download the image from the provided URL.'), 400

    try:
        with open(image_filename, 'rb') as f:
            image_bytes = f.read()
            image = Image.open(io.BytesIO(image_bytes))
    except UnidentifiedImageError:
        return jsonify(error='Unable to identify image file. Please provide a valid image.'), 400

    output_bytes = remove(image_bytes)

    return send_file(
        io.BytesIO(output_bytes),
        mimetype='image/png'
    )

if __name__ == '__main__':
    PORT = 5000
    app.run(debug=False, port=PORT)
