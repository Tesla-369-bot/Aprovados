import requests

# URL do arquivo no GitHub
url = "https://raw.githubusercontent.com/Tesla-369-bot/Aprovados/main/codigo.md"

# Faz a requisição ao arquivo no GitHub
response = requests.get(url)
conteudo = response.text

# Exibe todo o conteúdo
print(conteudo)
