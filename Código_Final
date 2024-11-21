#include <LiquidCrystal.h>

#define umidadeAnalog A0
#define pushbuttonOptions 7
#define pushbuttonDone 8
#define relePin 9

LiquidCrystal lcd(2, 3, 4, 5, 11, 12);

int humidityValue;
int nivelDeIrrigacao = 1;
bool configurandoNivel = false;
unsigned long ultimoTempoDeIrrigacao = 0;
unsigned long intervaloDeIrrigacao = 0;

// Definir os intervalos em milissegundos
#define INTERVALO_DIARIO 86400000
#define INTERVALO_2DIAS 172800000

void MenuLCD() {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Humidade: ");
    lcd.print(humidityValue);

    lcd.setCursor(0, 1);
    if (nivelDeIrrigacao == 1) {
        lcd.print("Frequencia: ALTA");
    } else if (nivelDeIrrigacao == 2) {
        lcd.print("Frequencia: DIARIA");
    } else if (nivelDeIrrigacao == 3) {
        lcd.print("Frequencia: 2 DIAS");
    }
}

void ConfigurandoMenu() {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Usar Frequencia: ");
    lcd.setCursor(0, 1);
    if (nivelDeIrrigacao == 1) {
        lcd.print("---> ALTA");
    } else if (nivelDeIrrigacao == 2) {
        lcd.print("---> DIARIA");
    } else if (nivelDeIrrigacao == 3) {
        lcd.print("---> 2 DIAS");
    }
}

void abrirValvula() {
    digitalWrite(relePin, HIGH);
}

void fecharValvula() {
    digitalWrite(relePin, LOW);
}

void abrirBombaRapido() {
    digitalWrite(relePin, LOW);
    unsigned long tempoBomba = millis();
    while (millis() - tempoBomba < 5000) {
        int buttonStateOptions = digitalRead(pushbuttonOptions);
        int buttonStateDone = digitalRead(pushbuttonDone);

        if (buttonStateOptions == LOW || buttonStateDone == LOW) {
            configurandoNivel = true;
            fecharValvula();
            return;
        }
        delay(100);
    }
    digitalWrite(relePin, HIGH);
    delay(1000);
}

void loop() {
    humidityValue = analogRead(umidadeAnalog);
    humidityValue = map(humidityValue, 1023, 315, 0, 100);

    // Leitura e alternancia do estado dos botoes
    int buttonStateOptions = digitalRead(pushbuttonOptions);
    static int lastButtonStateOptions = HIGH;
    if (buttonStateOptions == LOW && lastButtonStateOptions == HIGH) {
        configurandoNivel = !configurandoNivel;
        delay(200);
    }
    lastButtonStateOptions = buttonStateOptions;

    // Modo configuracao
    if (configurandoNivel) {
        ConfigurandoMenu();

        int buttonStateDone = digitalRead(pushbuttonDone);
        static int lastButtonStateDone = HIGH;
        if (buttonStateDone == LOW && lastButtonStateDone == HIGH) {
            nivelDeIrrigacao++;
            if (nivelDeIrrigacao > 3) {
                nivelDeIrrigacao = 1;
            }
            delay(200); // Debounce
        }
        lastButtonStateDone = buttonStateDone;

    } else {
        // Exibe menu LCD no modo padrao
        MenuLCD();

        unsigned long tempoAtual = millis();

        // Logica de irrigacao
        if (nivelDeIrrigacao == 1) { // Modo ALTA
            if (humidityValue < 40) {
                abrirBombaRapido();
            }
        } else if (nivelDeIrrigacao == 2) { // Modo DIARIA
            intervaloDeIrrigacao = INTERVALO_DIARIO;
            if ((tempoAtual - ultimoTempoDeIrrigacao >= intervaloDeIrrigacao) && humidityValue < 40) {
                abrirBombaRapido();
                ultimoTempoDeIrrigacao = tempoAtual;
            }
        } else if (nivelDeIrrigacao == 3) { // Modo CADA 2 DIAS
            intervaloDeIrrigacao = INTERVALO_2DIAS;
            if ((tempoAtual - ultimoTempoDeIrrigacao >= intervaloDeIrrigacao) && humidityValue < 40) {
                abrirBombaRapido();
                ultimoTempoDeIrrigacao = tempoAtual;
            }
        }
    }
    delay(100);
}
