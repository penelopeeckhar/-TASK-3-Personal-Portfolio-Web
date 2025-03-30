# -TASK-3-secure-coding-review
# ABIR MAJDI

from flask import Flask, request, jsonify
import subprocess
import shlex
import logging
import re

app = Flask(__name__)

#Configuration du journal de sécurité

logging.basicConfig(filename="security.log", level=logging.WARNING, 
                    format='%(asctime)s - %(levelname)s - %(message)s')

#Liste des commandes autorisées

ALLOWED_COMMANDS = {"ls", "whoami", "uptime"}

#Expression régulière pour valider les entrées

COMMAND_PATTERN = re.compile(r"^[a-z]+$")

@app.route('/run', methods=['GET'])
def run_command():
    cmd = request.args.get('cmd', '').strip()
    
    # Vérifier si la commande est autorisée
    if cmd not in ALLOWED_COMMANDS or not COMMAND_PATTERN.match(cmd):
        logging.warning(f"Tentative d'exécution de commande interdite : {cmd}")
        return jsonify({"error": "Commande non autorisée"}), 403
    
    try:
        # Exécution sécurisée avec timeout
        output = subprocess.check_output(shlex.split(cmd), stderr=subprocess.STDOUT, timeout=3)
        return jsonify({"output": output.decode().strip()})
    except subprocess.CalledProcessError:
        return jsonify({"error": "Erreur lors de l'exécution de la commande"}), 400
    except subprocess.TimeoutExpired:
        return jsonify({"error": "Commande trop longue"}), 408
    except Exception as e:
        logging.error(f"Erreur inattendue : {str(e)}")
        return jsonify({"error": "Une erreur est survenue"}), 500

if __name__ == '__main__':
    app.run(debug=False, host='0.0.0.0', port=5000)  # Debug désactivé, accessible sur le réseau

