# Practica-ChatGPT

**Código**
# #Jiménez Rivera Paulina 20211796
import json
import network
import time
import urequests
import ssd1306

# Internal libs
i2c = machine.I2C(0, sda=machine.Pin(0), scl=machine.Pin(1), freq=400000)
pantalla = ssd1306.SSD1306_I2C(128, 64, i2c)
cmd = machine.Pin(16, machine.Pin.IN, machine.Pin.PULL_UP)

def chatgp(identif, pswd, endpoint, api_key, model, prompt, max_tokens):
    """
        Description: This is a function to hit chat gpt api and get
            a response.
        
        Parameters:
        
        identif[str]: The name of your internet connection
        pswd[str]: Password for your internet connection
        endpoint[str]: API enpoint
        api_key[str]: API key for access
        model[str]: AI model (see openAI documentation)
        prompt[str]: Input to the model
        max_tokens[int]: The maximum number of tokens to
            generate in the completion.
        
        Returns: Simply prints the response
    """
    # Just making our internet connection
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(identif, pswd)
    
    # Wait for connect or fail
    max_wait = 10
    while max_wait > 0:
      if wlan.status() < 0 or wlan.status() >= 3:
        break
      max_wait -= 1
      print('Buscando conexión. ¡Espere!')
      time.sleep(1)
    # Handle connection error
    if wlan.status() != 3:
       print(wlan.status())
       raise RuntimeError('La conexión ha fallado. Intente más tarde.')
    else:
      print('Conexión exitosa')
      print(wlan.status())
      status = wlan.ifconfig()
    
    ## Begin formatting request
    headers = {'Content-Type': 'application/json',
               "Authorization": "Bearer " + api_key}
    data = {"model": model,
            "prompt": prompt,
            "max_tokens": max_tokens}
    
    print("Prompt en proceso de envío")
    r = urequests.post("https://api.openai.com/v1/{}".format(endpoint),
                       json=data,
                       headers=headers)
    
    if r.status_code >= 300 or r.status_code < 200:
        print("Lo siento. Hubo un error en generar la respuesta \n" +
              "Estado de la solicitud: " + str(r.text))
    else:
        print("Petición exitosa")
        response_data = json.loads(r.text)
        completion = response_data["choices"][0]["text"]
        print(completion)
        
        pantalla.fill(0)

        completion_text = completion
        max_chars_per_line = 15
        lines = [completion_text[i:i+max_chars_per_line] for i in range(0, len(completion_text), max_chars_per_line)]

        for i, line in enumerate(lines):
            pantalla.text(line, 0, i * 10)  # Usar un espaciado vertical de 10 píxeles

        pantalla.show()

    r.close()

def boton(p):
    if not cmd.value():
        print("Enviando prompt... Botón presionado")
        # Mostrar "Sending prompt" en la pantalla OLED
        pantalla.fill(0)  # Borra la pantalla
        pantalla.text("Solicitud enviada", 0, 0)
        pantalla.show()
        
        
        chatgp("Jesail",
                 "",
                 "completions",
                 "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",
                 "text-davinci-003",
                  "Give me the name of a random country",
                 20)

# Configura una interrupción en el botón para llamar a button_callback cuando se presiona el botón
cmd.irq(trigger=machine.Pin.IRQ_FALLING, handler=boton)

while True:
    # Tu código principal puede continuar aquí
    pass
