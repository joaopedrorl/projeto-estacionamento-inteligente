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

const byte PINO_SERVO = 9;
const byte TRIG_ENTRADA = 7, ECHO_ENTRADA = 6;
const byte TRIG_VAGA = 4,    ECHO_VAGA = 3;
const byte LED_VERDE = 11,   LED_VERMELHO = 10;
const byte PINO_BUZZER = 8;

const int  ANGULO_FECHADA = 0;
const int  ANGULO_ABERTA  = 90;
const long DIST_ENTRADA_CM = 10;
const long DIST_VAGA_CM    = 8;
const unsigned long TEMPO_ABERTA_MS = 3000;

Servo cancela;
bool cancelaAberta = false;
unsigned long ultimaDeteccao = 0;

long lerDistancia(byte trig, byte echo) {
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);
  unsigned long duracao = pulseIn(echo, HIGH, 30000UL);
  if (duracao == 0) return -1;   // sem eco
  return duracao / 58;           // em cm
}

// Mediana de 3 leituras: descarta leituras falsas
long lerFiltrado(byte trig, byte echo) {
  long a = lerDistancia(trig, echo); delay(10);
  long b = lerDistancia(trig, echo); delay(10);
  long c = lerDistancia(trig, echo);
  return max(min(a, b), min(max(a, b), c));
}

void setup() {
  Serial.begin(9600);
  cancela.attach(PINO_SERVO);
  cancela.write(ANGULO_FECHADA);
  pinMode(TRIG_ENTRADA, OUTPUT); pinMode(ECHO_ENTRADA, INPUT);
  pinMode(TRIG_VAGA, OUTPUT);    pinMode(ECHO_VAGA, INPUT);
  pinMode(LED_VERDE, OUTPUT);    pinMode(LED_VERMELHO, OUTPUT);
  pinMode(PINO_BUZZER, OUTPUT);
}

void loop() {
  unsigned long agora = millis();

  // Entrada: abre ao detectar e só fecha 3 s depois que o carro sair
  long dEntrada = lerFiltrado(TRIG_ENTRADA, ECHO_ENTRADA);
  bool carroNaEntrada = dEntrada > 0 && dEntrada < DIST_ENTRADA_CM;

  if (carroNaEntrada) {
    ultimaDeteccao = agora;
    if (!cancelaAberta) {
      cancela.write(ANGULO_ABERTA);
      tone(PINO_BUZZER, 1500, 200);   // bip curto, não trava
      cancelaAberta = true;
    }
  } else if (cancelaAberta && agora - ultimaDeteccao >= TEMPO_ABERTA_MS) {
    cancela.write(ANGULO_FECHADA);
    cancelaAberta = false;
  }

  delay(30);  // evita interferência entre os sensores

  // Vaga
  long dVaga = lerFiltrado(TRIG_VAGA, ECHO_VAGA);
  bool ocupada = dVaga > 0 && dVaga < DIST_VAGA_CM;
  digitalWrite(LED_VERMELHO, ocupada);
  digitalWrite(LED_VERDE, !ocupada);

  delay(50);
}
