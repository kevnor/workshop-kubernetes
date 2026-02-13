from http.server import SimpleHTTPRequestHandler, HTTPServer

PORT = 8000

Handler = SimpleHTTPRequestHandler

with HTTPServer(("", PORT), Handler) as httpd:
    print(f"Server running on port {PORT}")
    httpd.serve_forever()
