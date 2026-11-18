# projeto-estacionamento-inteligente
Maquete de estacionamento inteligente com Arduino, sensor ultrassônico, servo motor e LEDs.
# Projeto Estacionamento Inteligente

## 1. Descrição do Problema
O projeto visa resolver a dificuldade de controle de fluxo e monitoramento de vagas em estacionamentos de pequeno porte. Através de automação com microcontrolador, o sistema automatiza a abertura da cancela de entrada e sinaliza em tempo real se a vaga está livre ou ocupada, otimizando o espaço e agilizandO o acesso.

## 2. Lista de Componentes
* 1x Placa Arduino Uno
* 2x Sensores Ultrassônicos (HC-SR04)
* 2x Servo Motores SG90 (sendo um usado na cancela)
* 1x LED Verde (Indicador de vaga livre)
* 1x LED Vermelho (Indicador de vaga ocupada)
* 2x Resistores de 220Ω
* 1x Buzzer
* Fios Jumpers e Protoboard
* Maquete estruturada em MDF

## 3. Esquema de Ligação (Pinout)
* **Servo Motor (Cancela):** Pino 9
* **Sensor Ultrassônico (Entrada):** Trig pino 7, Echo pino 6
* **Sensor Ultrassônico (Vaga):** Trig pino 4, Echo pino 3
* **LED Verde:** Pino 11 (com resistor de 220Ω)
* **LED Vermelho:** Pino 10 (com resistor de 220Ω)
* **Buzzer:** Pino 8
* **Alimentação:** 5V e GND conectados aos trilhos da protoboard.

## 4. Instruções de Uso
1. Conecte o Arduino Uno à porta USB do computador.
2. Certifique-se de que todas as ligações físicas e a alimentação externa dos motores/sensores estão corretas.
3. Carregue o código-fonte principal para a placa através da Arduino IDE.
4. O sistema iniciará automaticamente: a aproximação de um veículo no sensor de entrada acionará a abertura da cancela, enquanto o sensor de vaga alternará o estado dos LEDs conforme a ocupação.


#include <Servo.h>


#define PINO_SERVO 9


#define TRIG_ENTRADA 7
#define ECHO_ENTRADA 6


#define TRIG_VAGA 4
#define ECHO_VAGA 3


#define LED_VERDE 11
#define LED_VERMELHO 10

// Buzzer de alerta
#define PINO_BUZZER 8

Servo cancela; 

void setup() {
  Serial.begin(9600);

  cancela.attach(PINO_SERVO);
  cancela.write(0);

  pinMode(TRIG_ENTRADA, OUTPUT);
  pinMode(ECHO_ENTRADA, INPUT);
  
  pinMode(TRIG_VAGA, OUTPUT);
  pinMode(ECHO_VAGA, INPUT);

  // Configuração dos LEDs e Buzzer
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);
  pinMode(PINO_BUZZER, OUTPUT);
}

void loop() {

  long distanciaEntrada = lerDistancia(TRIG_ENTRADA, ECHO_ENTRADA);

  if (distanciaEntrada > 0 && distanciaEntrada < 10) {
    cancela.write(90);          // Abre a cancela (90 graus)
    tone(PINO_BUZZER, 1500);    // Emite som no buzzer
    delay(3000);                // Mantém aberta por 3 segundos
    noTone(PINO_BUZZER);        // Desliga o buzzer
    cancela.write(0);           // Fecha a cancela novamente
  }


  long distanciaVaga = lerDistancia(TRIG_VAGA, ECHO_VAGA);

  // Se houver um carro estacionado a menos de 8 cm no fundo da vaga
  if (distanciaVaga > 0 && distanciaVaga < 8) {
    digitalWrite(LED_VERMELHO, HIGH); // Vaga ocupada
    digitalWrite(LED_VERDE, LOW);     
  } else {
    digitalWrite(LED_VERMELHO, LOW);  
    digitalWrite(LED_VERDE, HIGH);    // Vaga livre
  }

  delay(200); // Pequena pausa para estabilizar as leituras
}

// Função padrão para calcular a distância com o sensor ultrassônico
long lerDistancia(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duracao = pulseIn(echoPin, HIGH, 30000); // Timeout de segurança
  long distancia = duracao * 0.034 / 2;        // Conversão para centímetros
  return distancia;
}

