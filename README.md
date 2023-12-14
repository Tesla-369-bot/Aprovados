```python



token = '6286275485:AAHaK-uRfNSr-hSCH0xEv3iu4tSyExa5vgI'
chat_id = '-1001555213923'

async def send_message(token: str, chat_id: str, text: str):
    bot = Bot(token=token)
    await bot.send_message(chat_id=chat_id, text=text)

def send_message_now(token: str, chat_id: str, messages: str):
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(send_message(token, chat_id, messages))

def speak_text(text):
    engine = pyttsx3.init()
    engine.setProperty('rate', 220)  # Ajuste a taxa de fala
    engine.setProperty('volume', 1.0)  # Ajuste o volume entre 0.0 e 1.0
    engine.say(text)
    engine.runAndWait()

def stop(lucro, gain, loss):
    lucro_str = str(round(lucro, 2)).rstrip('0').rstrip('.')
    if lucro <= float('-' + str(abs(loss))):
        speak_text("Stop loss batido. Sua perda foi de " + lucro_str + ". Não foi dessa vez. Volte apenas amanhã!")
        #caminho_programa_exe = r"C:\Users\Tiffany Bit\Desktop\Nova pasta (3)\2024\dist\LigarBot.ba"
        #print("Iniciando o programa...")
        #subprocess.run([caminho_programa_exe])
    if lucro >= float(abs(gain)):
        speak_text("Stop gain batido. Seu lucro foi de " + lucro_str + ". Parabéns. Volte apenas amanhã!")
        #caminho_programa_exe = r"C:\Users\Tiffany Bit\Desktop\Nova pasta (3)\2024\dist\LigarBot.bat"
        #print("Iniciando o programa...")
        #subprocess.run([caminho_programa_exe])
def Martingale(valor, payout):
    lucro_esperado = valor * payout
    perca = float(valor)    

    while True:
        if round(valor * payout, 2) > round(abs(perca) + lucro_esperado, 2):
            return round(valor, 2)
            break
        valor += 0.01

def Payout(API, par):
    API.subscribe_strike_list(par, 1)
    while True:
        d = API.get_digital_current_profit(par, 1)
        if d != False:
            d = round(int(d) / 100, 2)
            break
        time.sleep(1)
    API.unsubscribe_strike_list(par, 1)

    return d

print('''
        
 ------------------------------------
''')

email = 'pandoraakio77@gmail.com'#input(' Coloque seu email: ')
senha = '977404akio' #input(' Coloque sua senha: ')
conta = 'PRACTICE' #input(' Conta PRACTICE ou REAL: ')
API = IQ_Option(email, senha)
API.connect()

API.change_balance(conta) # PRACTICE / REAL

if API.check_connect():
    print(' Conectado com sucesso!')
else:
    print(' Erro ao conectar')
    input('\n\n Aperte enter para sair')
    sys.exit()

par = 'EURUSD' #input(' Indique uma paridade para operar: ').upper()
valor_entrada = float(input(' Indique um valor para entrar: '))
valor_entrada_b = float(valor_entrada)

martingale = 1
martingale += 1

stop_loss = float(input(' Indique o valor de Stop Loss: '))
stop_gain = 0

lucro = 0
payout = Payout(par)
#messages = ('ANALISES INICIADA EM : '  + par + ' BORA FAZER GRANA!!!')
#send_message_now(token, chat_id, messages)

while True:
    
    agora = datetime.now()
    minutos = agora.second
    entrar = True if minutos % 55 == 0 else False

    if entrar:
        speak_text("Verificando oportunidade no quadrante")
        dir = False

        velas = API.get_candles(par, 300, 4, time.time())

        for i, vela in enumerate(velas):
            velas[i] = 'g' if vela['open'] < vela['close'] else 'r' if vela['open'] > vela['close'] else 'd'

        cores = ' '.join(velas)
        cores = velas[0] + '1 ' + velas[1] + '2 ' + velas[2] + '3 ' + velas[3] + '4'

        if velas[3] == 'g' and cores.count('d') == 0:
            dir = 'put'
        if velas[3] == 'r' and cores.count('d') == 0:
            dir = 'call'

        if dir:
            print('Direção:', dir)

            valor_entrada = valor_entrada_b
            for i in range(martingale):

                status, id = API.buy_digital_spot(par, valor_entrada, dir, 1)
                if dir == 'call':
                    speak_text("Operação de compra")
                if dir == 'put':
                    speak_text("Operação de venda")

                if status:
                    while True:
                        status, valor = API.check_win_digital_v2(id)

                        if status:
                            valor = valor if valor > 0 else float('-' + str(abs(valor_entrada)))
                            lucro += round(valor, 2)

                            print('Resultado operação: ', end='')
                            print('WIN /' if valor > 0 else 'LOSS /' , round(valor, 2), '/', round(lucro, 2),('/ '+str(i)+ ' GALE' if i > 0 else '' ))

                            valor_entrada = Martingale(valor_entrada, payout)

                            stop(lucro, stop_gain, stop_loss)

                            break

                    if valor > 0 : break

                else:
                    print('\nERRO AO REALIZAR OPERAÇÃO\n\n')

    time.sleep(0.5)



```
