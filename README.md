# LAB 2 – Estimación del nivel de estrés basada en GSR (EDA)

Este repositorio desarrolla el Laboratorio 2, cuyo objetivo fue diseñar e implementar un biosensor capaz de medir y analizar la respuesta galvánica cutánea (GSR/EDA), con el fin de estimar un nivel de estrés en tiempo real. Los datos adquiridos se transmiten de forma inalámbrica hacia MATLAB para su visualización y procesamiento.

## Integrantes
- Jhonathan David Guevara
- Juan Pablo Díaz 
- María Paula Fernández

## 1. Introducción

La actividad electrodérmica (EDA, por sus siglas en inglés) agrupa los fenómenos eléctricos asociados a la piel, especialmente las variaciones en su conductancia originadas por la actividad de las glándulas sudoríparas y la modulación del sistema nervioso simpático. Dentro de este contexto, la respuesta galvánica cutánea (GSR) es una de las variables fisiológicas más utilizadas para evaluar cambios de activación autonómica frente a estímulos emocionales, cognitivos o sensoriales. Por su naturaleza no invasiva, facilidad de instrumentación y sensibilidad a cambios fisiológicos, la GSR es ampliamente empleada en ingeniería biomédica, psicofisiología y estudios de interacción humano-máquina.
La guía de laboratorio establece que el estudiante debe construir un dispositivo vestible capaz de capturar variaciones de la GSR y utilizarlo para el monitoreo del estrés durante la resolución de tareas cognitivas. De igual manera, solicita identificar la componente estacionaria o basal y la componente transitoria de la señal, justificar el diseño eléctrico, garantizar la seguridad del usuario y documentar las limitaciones del sistema. En ese marco, el presente informe organiza y consolida la implementación realizada por el grupo, integrando los fundamentos fisiológicos, el diseño del circuito, el procedimiento experimental, el análisis de la señal y la discusión técnica de los resultados.

## 2. Fundamento teórico (EDA / GSR)

## 2.1 Actividad electrodérmica y respuesta galvánica cutánea

La EDA describe los cambios eléctricos de la piel relacionados con la activación simpática. Cuando una persona experimenta un estímulo interno o externo, aumenta la actividad de las glándulas sudoríparas ecrinas, principalmente en palmas y dedos, lo que incrementa la conductancia cutánea. En la literatura se distinguen dos componentes de interés. La primera es la SCL (Skin Conductance Level), correspondiente al nivel tónico o basal de la señal y asociada al estado general de activación. La segunda es la SCR (Skin Conductance Response), que representa respuestas fásicas o transitorias, caracterizadas por ascensos relativamente rápidos seguidos de una recuperación más lenta.

## 2.2 Relación entre GSR y estrés

Aunque la GSR no constituye por sí sola una medida específica de “estrés”, sí es un marcador sensible de activación simpática. Por ello, en condiciones controladas permite comparar estados como reposo, carga mental, verbalización o respuesta a estímulos fisiológicos, por ejemplo, una inspiración profunda. Su principal fortaleza radica en que cambios pequeños en la activación autonómica producen modificaciones detectables en la conductancia de la piel. Su principal limitación es que numerosos factores no emocionales también influyen sobre la señal, entre ellos el movimiento, la presión de los electrodos, la temperatura ambiente, la hidratación cutánea y la variabilidad intersujeto.


## 3. Objetivos

### 3.1 Objetivo general
Implementar un sistema continuo de medición de GSR para estimar el nivel de estrés.

### 3.2 Objetivos específicos
- Adquirir la señal GSR con un sistema vestible estable.
- Garantizar seguridad del usuario limitando la corriente a través de la piel.
- Transmitir la señal de forma inalámbrica y visualizarla en MATLAB en tiempo real.
- Proponer un criterio de clasificación (bajo/medio/alto) con umbrales por sujeto.
- Comparar el comportamiento de la señal en reposo, respiración profunda, movimiento y tarea cognitiva.

## 4. Materiales y herramientas

### 4.1 Hardware
- ESP32 (ADC1 para adquisición)
- 2 electrodos metálicos (monedas)
- Resistencia: 100 kΩ
- Capacitor: 5 µF
- Cables, protoboard o montaje equivalente
- Elementos de sujeción (cinta Micropore /velcro/elástico)

### 4.2 Software

- MATLAB (lectura del flujo de datos, graficación y análisis)
- Arduino IDE
  
## 5 Diseño del sistema y acondicionamiento de la señal

## 5.1 Criterio de seguridad eléctrica
La guía exige demostrar que, aun en el peor caso, la corriente a través del sujeto no excede 1 mA. 
La seguridad se evalúa con el caso extremo de resistencia de piel cercana a 0 Ω, donde la corriente máxima queda limitada por la resistencia serie:

$$
I_{\text{max}}=\frac{V}{R}=\frac{3.3\,\text{V}}{100\,\text{k}\Omega}=3.3\times10^{-5}\,\text{A}=0.033\,\text{mA}
$$

Este valor es inferior a 1 mA, cumpliendo el criterio de seguridad planteado para el laboratorio.

### 5.2 Conversión de resistencia de piel a voltaje

El circuito se implementó como un divisor resistivo: 3.3 V → R fija (100 kΩ) → nodo de medición (ADC) → Rskin → GND.
Adicionalmente, se conectó un capacitor de 5 µF entre el nodo de medición y tierra para suavizar variaciones rápidas. Bajo esta configuración, el voltaje leído por el ADC está dado por Vadc = Vcc · Rskin / (R + Rskin). 
Por tanto, la resistencia equivalente de la piel puede estimarse como Rskin = R · Vadc / (Vcc − Vadc), y la conductancia como Gskin = 1 / Rskin = (Vcc − Vadc) / (R · Vadc).
Esta relación es importante porque, en la topología utilizada, un aumento de la conductancia cutánea implica una disminución de Rskin y, en consecuencia, una disminución del voltaje medido en el ADC. Por ello, si el análisis se realiza directamente con voltaje, debe aclararse que la señal es inversamente proporcional a la conductancia. Para una interpretación fisiológica más rigurosa, se recomienda transformar el voltaje a conductancia estimada antes de aplicar la clasificación del nivel de estrés.

Esquema conceptual:
- 3.3 V → R (100 kΩ) → nodo (ADC1) → R_skin (piel entre electrodos) → GND
- C (5 µF) desde el nodo (ADC1) a GND

### 6.2 Comportamiento del filtro RC

$$
\tau = R\,C = (100\,000)\,(5\times10^{-6}) = 5\times10^{-1}\,\text{s} = 0.5\,\text{s}
$$

$$
f_c = \frac{1}{2\pi \tau} \approx \frac{1}{2\pi(0.5)} \approx 0.318\,\text{Hz}
$$

Con estos valores, el capacitor implementa un **filtro pasa-bajas** que ayuda a suavizar ruido y variaciones rápidas, dejando principalmente la componente lenta típica de GSR (SCL). La separación SCL/SCR puede realizarse adicionalmente mediante filtrado digital en MATLAB (promedio móvil o filtros pasa-bajas/pasa-altas suaves).

### 6.3 Selección anatómica y fijación de electrodos 

Se eligieron los dedos como región anatómica de medición debido a su alta densidad de glándulas sudoríparas y su respuesta electrodérmica pronunciada. La fijación se realizó procurando una presión uniforme, sin comprimir excesivamente el tejido, y asegurando que el contacto eléctrico se mantuviera estable. También se redujo el movimiento relativo entre cables y electrodos para disminuir artefactos mecánicos durante la adquisición.

## 7.Metodología experimental

## 7.1.Preparación y diseño
Inicialmente se realizó una revisión bibliográfica sobre EDA, SCL y SCR, seguida del análisis del criterio de seguridad eléctrica. Posteriormente se diseñó el circuito de medición y se verificó el comportamiento del nodo de salida en condiciones de reposo. Finalmente se definió el montaje vestible, la región anatómica de captura y la forma de fijación de los electrodos para minimizar interferencias.


## 7.2.Región anatómica y colocación de electrodos
Se utilizaron dedos (falanges distales) por su buena respuesta electrodérmica y facilidad de montaje. Para reducir artefactos:
- Asegurar contacto uniforme sin presión excesiva.
- Minimizar movimiento de los cables.
- Mantener la piel limpia y seca antes de la medición.

## 7.3. Adquisición de datos
Con el sistema ensamblado, que se observa en la imagen 1:
<img width="517" height="693" alt="image" src="https://github.com/user-attachments/assets/a356a539-c040-41e7-9cee-b181e16cdfbe" />

Imagen 1.

Se adquirió la señal GSR en tiempo real mientras el sujeto permanecía en reposo. Después se realizó una maniobra de inspiración profunda seguida de exhalación lenta, buscando generar una respuesta transitoria visible. También se registró la señal durante movimiento, escritura o caminata para evaluar la estabilidad del vestible y reconocer la aparición de artefactos. Con base en la señal basal y en la respuesta obtenida durante la maniobra respiratoria, se propuso un esquema de umbrales para clasificar el nivel de activación.

## 8.Configuración del sistema

### 8.1 Configuración de muestreo

- ADC: ADC1 del ESP32
- Frecuencia de muestreo: 50 Hz
- Transmisión: inalámbrica hacia MATLAB por medio de ARDUINO IDE con el siguiente código:

```
#include "BluetoothSerial.h"
BluetoothSerial SerialBT;

// Recomendado: ADC1 (no se pelea con WiFi)
const int ADC_PIN = 34;

volatile int fs_hz = 50;
bool streaming = false;          
bool lastClient = false;

// Buffer para comandos sin bloqueo
String cmdLine = "";

uint32_t nextSendMs = 0;

static inline int adcAvg(int pin, int n) {
  long acc = 0;
  for (int i = 0; i < n; i++) acc += analogRead(pin);
  return (int)(acc / n);
}

void sendSample() {
  uint32_t t = millis();

  // promedio para estabilizar (ajusta 4/8/16)
  int raw = adcAvg(ADC_PIN, 8);

  int mv  = analogReadMilliVolts(ADC_PIN);

  // t_ms,raw,mV
  SerialBT.printf("%lu,%d,%d\n", (unsigned long)t, raw, mv);
}

void handleCommand(const String &c) {
  String cmd = c;
  cmd.trim();
  cmd.toUpperCase();

  if (cmd == "PING") {
    SerialBT.println("PONG");
    return;
  }
  if (cmd == "START") {
    streaming = true;
    SerialBT.println("OK START");
    return;
  }
  if (cmd == "STOP") {
    streaming = false;
    SerialBT.println("OK STOP");
    return;
  }
  if (cmd == "READ") {
    sendSample();
    return;
  }
  if (cmd.startsWith("RATE")) {
    // Ej: "RATE 100"
    int sp = cmd.indexOf(' ');
    if (sp > 0) {
      int v = cmd.substring(sp + 1).toInt();
      if (v >= 1 && v <= 500) {
        fs_hz = v;
        SerialBT.printf("OK RATE %d\n", fs_hz);
      } else {
        SerialBT.println("ERR RATE (1..500)");
      }
    } else {
      SerialBT.println("ERR RATE format: RATE 50");
    }
    return;
  }

  SerialBT.println("ERR CMD");
}

void setup() {
  Serial.begin(115200);

  SerialBT.begin("ESP32_ADC2");
  Serial.println("Listo. Empareja 'ESP32_ADC2'.");

  // ADC config
  analogReadResolution(12);
  analogSetPinAttenuation(ADC_PIN, ADC_11db);
  pinMode(ADC_PIN, INPUT);

  nextSendMs = millis();
}

void loop() {
  bool has = SerialBT.hasClient();

  // Detectar conexión/desconexión
  if (has != lastClient) {
    lastClient = has;
    if (has) {
      streaming = false;                 
      cmdLine = "";
      SerialBT.println("READY");        
      Serial.println(" Cliente BT conectado");
    } else {
      streaming = false;
      Serial.println(" Cliente BT desconectado");
    }
  }

  // Leer comandos SIN BLOQUEO (carácter a carácter)
  while (SerialBT.available()) {
    char ch = (char)SerialBT.read();
    if (ch == '\r') continue;
    if (ch == '\n') {
      if (cmdLine.length() > 0) handleCommand(cmdLine);
      cmdLine = "";
    } else {
      // evita que crezca infinito si llega basura
      if (cmdLine.length() < 80) cmdLine += ch;
      else cmdLine = "";
    }
  }
  if (has && streaming) {
    uint32_t now = millis();
    uint32_t period = (fs_hz > 0) ? (1000UL / (uint32_t)fs_hz) : 20UL;

    if ((int32_t)(now - nextSendMs) >= 0) {
      sendSample();
      nextSendMs = now + period;
    }
  }
}
````
### 8.2 Datos transmitidos a MATLAB
Flujo de trabajo:
1. Conexión inalámbrica y lectura continua de muestras.
2. Graficación en tiempo real de la señal.
3. Filtrado digital:
   - Promedio móvil (suavizado)
   - Pasa-bajas para resaltar SCL
   - Derivada o pasa-altas suave para resaltar SCR
4. Cálculo del nivel (bajo/medio/alto) por ventanas temporales.

````
COM = "COM8";
baud = 115200;

fs   = 50;        % Hz 
dur  = 120;       % segundos
ventana = 20;     % segundos visibles (scroll). 
% ===== Parámetros de filtrado =====
fc_suavizado = 2;     
orden_suave  = 2;

fc_tonica    = 0.05;  % Hz -> componente estacionaria (muy lenta)
orden_tonica = 2;

% ===== Umbrales =====
th_moderado = 2000;   % >= 2000 -> estrés medio/moderado
th_alto     = 2500;   % >= 2500 -> estrés super alto

% ===== Conexión Serial BT =====
s = serialport(COM, baud, "Timeout", 1);
configureTerminator(s, "LF");
flush(s);

% Handshake
writeline(s, "PING");
resp = readline(s);
disp("PING-> " + resp);

% Configurar muestreo y arrancar
writeline(s, "RATE " + fs);  disp(readline(s));   % "OK RATE ..."
writeline(s, "START");       disp(readline(s));   % "OK START"

% ===== Diseñar filtros =====
% Suavizado (tiempo real): filter() con estado
[bS,aS] = butter(orden_suave, fc_suavizado/(fs/2));
ziS = zeros(max(length(aS),length(bS))-1,1);

% Tónica (tiempo real)
[bT,aT] = butter(orden_tonica, fc_tonica/(fs/2));
ziT = zeros(max(length(aT),length(bT))-1,1);
N = dur * fs;
t_s   = nan(N,1);
t_ms  = nan(N,1);
raw   = nan(N,1);
mv    = nan(N,1);

mv_suave  = nan(N,1);   % señal filtrada (suave)
tonica    = nan(N,1);   % componente estacionaria
fasica    = nan(N,1);   % componente transitoria (aprox)

stress_code  = nan(N,1);      % 0 normal, 1 moderado, 2 alto
stress_label = strings(N,1);  % texto

% ===== Figuras en tiempo real =====
figure;

subplot(2,1,1);
h_rawmv   = animatedline; hold on;
h_suave   = animatedline;
grid on;
xlabel("Tiempo (s)");
ylabel("mV");
title("ESP32 GSR/ADC por BT (cruda vs suavizada)");
legend("Cruda (mV)","Suavizada (mV)");

subplot(2,1,2);
h_tonica = animatedline; hold on;
h_fasica = animatedline;
grid on;
xlabel("Tiempo (s)");
ylabel("mV");
title("Componentes: tónica (estacionaria) y fásica (transitoria)");
legend("Tónica","Fásica");

% Cajita de estado arriba
ann = annotation('textbox', [0.12 0.94 0.78 0.05], ...
    'String', "Estado: ---", 'EdgeColor', 'none', ...
    'FontSize', 12, 'FontWeight', 'bold');

t0 = tic;
k = 0;
ultimoDraw = 0;

while toc(t0) < dur
    if s.NumBytesAvailable > 0
        line = readline(s);                 % "t_ms,raw,mV"
        p = split(strtrim(line), ",");

        if numel(p) >= 3
            k = k + 1;
            if k > N, break; end

            t_ms(k) = str2double(p(1));
            raw(k)  = str2double(p(2));
            mv(k)   = str2double(p(3));
            t_s(k)  = toc(t0);

            % Si llega algo raro, lo saltamos
            if any(isnan([t_ms(k), raw(k), mv(k)]))
                continue;
            end

            % ===== Filtros en tiempo real =====
            [mv_suave(k), ziS] = filter(bS, aS, mv(k), ziS);
            [tonica(k),   ziT] = filter(bT, aT, mv_suave(k), ziT);

            % Fásica = (señal suavizada - tónica)
            fasica(k) = mv_suave(k) - tonica(k);

            % ===== Clasificación por umbrales (usando RAW) =====
            if raw(k) >= th_alto
                stress_code(k)  = 2;
                stress_label(k) = "ESTRÉS SUPER ALTO";
            elseif raw(k) >= th_moderado
                stress_code(k)  = 1;
                stress_label(k) = "ESTRÉS MEDIO / MODERADO";
            else
                stress_code(k)  = 0;
                stress_label(k) = "NORMAL / TRANQUILO";
            end

            % ===== Dibujar =====
            subplot(2,1,1);
            addpoints(h_rawmv, t_s(k), mv(k));
            addpoints(h_suave, t_s(k), mv_suave(k));

            subplot(2,1,2);
            addpoints(h_tonica, t_s(k), tonica(k));
            addpoints(h_fasica, t_s(k), fasica(k));

            % Dibujar sin matar rendimiento (cada ~0.1 s)
            if (t_s(k) - ultimoDraw) > 0.1
                if ventana < dur
                    xl = [max(0, t_s(k)-ventana)  t_s(k)+0.1];
                else
                    xl = [0 dur];
                end
                subplot(2,1,1); xlim(xl);
                subplot(2,1,2); xlim(xl);

                % Actualizar estado (raw y etiqueta)
                ann.String = "Estado: " + stress_label(k) + ...
                    " | raw=" + raw(k) + " | mV=" + round(mv_suave(k), 1);

                drawnow limitrate;
                ultimoDraw = t_s(k);
            end
        end
    else
        pause(0.005);
    end
end

% ===== Parar streaming =====
writeline(s, "STOP");
if s.NumBytesAvailable > 0
    disp(readline(s)); % "OK STOP" si llega
end
clear s;

% ===== Recortar a lo recibido =====
t_s         = t_s(1:k);
t_ms        = t_ms(1:k);
raw         = raw(1:k);
mv          = mv(1:k);
mv_suave    = mv_suave(1:k);
tonica      = tonica(1:k);
fasica      = fasica(1:k);
stress_code = stress_code(1:k);
stress_label= stress_label(1:k);
````
Los umbrales definidos fueron: menor o igual a 2000 para estrés moderado y 2500 para estrés alto.

## 9. Resultados

- Señal en reposo (baseline).
  
<img width="408" height="200" alt="image" src="https://github.com/user-attachments/assets/1609355c-6181-42ee-b67f-e54820167a33" />

Imagen 2.

- Respuesta durante inspiración profunda (SCR visible).
  
  <img width="408" height="200" alt="image" src="https://github.com/user-attachments/assets/cfb4ed44-3085-4a02-9af7-dd59c4b8875e" />
  
Imagen 3.

## 10. Análisis

## 10.1 Eficacia del sistema para monitoreo ambulatorio
En oficinas, aulas universitarias y el hogar, un sistema de monitoreo basado en GSR puede ser útil como indicador complementario del nivel de activación autonómica, debido a su bajo costo, carácter no invasivo y relativa facilidad de integración con dispositivos embebidos. El vestible desarrollado permite capturar cambios fisiológicos en tiempo real y transmitirlos a una interfaz computacional, lo cual abre la posibilidad de usarlo en contextos de seguimiento, biofeedback o experimentación controlada. Sin embargo, su eficacia depende críticamente de la calidad del contacto electrodo-piel, de la reducción de artefactos por movimiento y de una correcta calibración por sujeto. En condiciones dinámicas o poco controladas, la confiabilidad disminuye si el sistema se usa como único indicador.

## 10.2 Alcance y limitaciones para estrés neonatal

La extrapolación del sistema desarrollado a la detección de estrés neonatal presenta limitaciones importantes. En primer lugar, la piel del recién nacido posee propiedades electrofisiológicas y biomecánicas distintas a las de un adulto, por lo que los materiales, la presión de contacto y la corriente aplicada deben reevaluarse cuidadosamente. En segundo lugar, los neonatos muestran gran sensibilidad a cambios térmicos, manejo clínico y movimientos involuntarios, factores que pueden modificar la señal y dificultar su interpretación. Por tanto, aunque el principio de medición autonómica es relevante desde el punto de vista fisiológico, un sistema como el construido en esta práctica no puede trasladarse directamente al ámbito neonatal sin rediseño del hardware, validación ética, materiales clínicamente aprobados y comparación con otros indicadores fisiológicos.

## 11. Preguntas propuestas

### 11.1 ¿Por qué una inspiración profunda incrementa la GSR?
Una inspiración profunda puede producir una activación autonómica transitoria asociada a cambios en el balance simpático-parasimpático y en la actividad de las glándulas sudoríparas. Como resultado, aumenta la conductancia eléctrica de la piel y aparece una respuesta transitoria tipo SCR. En la práctica, esto se observa como un cambio brusco respecto al nivel basal, seguido de una recuperación más lenta.

## 11.2 Ventajas y desventajas de la GSR como indicador de estrés
Entre las ventajas se encuentran su carácter no invasivo, bajo costo, sensibilidad a cambios de activación simpática y facilidad de instrumentación. Además, permite realizar comparaciones claras entre estados como reposo y carga mental. Entre las desventajas destacan su baja especificidad, ya que la señal también responde a temperatura, dolor, movimiento o cambios en el contacto de los electrodos; la necesidad de calibración individual; y la imposibilidad de utilizarla por sí sola como herramienta diagnóstica concluyente.

## 12. Conclusiones

Se logró desarrollar una propuesta funcional de biosensor vestible para la medición continua de la respuesta galvánica cutánea y su visualización inalámbrica en MATLAB. El diseño eléctrico cumplió con el criterio de seguridad del laboratorio, al limitar la corriente máxima por debajo de 1 mA incluso en el peor caso teórico. Desde el punto de vista fisiológico, la práctica permitió reconocer la diferencia entre la componente tónica y la componente fásica de la señal, así como discutir su relación con la activación simpática.
La GSR resultó ser una variable útil para comparar estados del mismo sujeto bajo distintas condiciones experimentales; sin embargo, también se evidenció que no es un marcador específico de estrés y que su interpretación depende del contexto, de la calidad del montaje y del procesamiento aplicado. En consecuencia, la fiabilidad del sistema es adecuada para fines académicos, demostrativos y de monitoreo relativo, pero debe entenderse con cautela cuando se pretende extrapolar a aplicaciones clínicas o diagnósticas.

## 13. Referencias

Boucsein, W. (2012). Electrodermal activity (2nd ed.). Springer.

Figner, B., & Murphy, R. O. (2011). Using skin conductance in judgment and decision making research. In M. Schulte-Mecklenbeck, A. Kuehberger, & R. Ranyard (Eds.), A handbook of process tracing methods for decision research: A critical review and user’s guide (pp. 163–184). Psychology Press.

Loggia, M. L., Juneau, M., & Bushnell, M. C. (2011). Autonomic responses to heat pain: Heart rate, skin conductance, and their relation to verbal ratings and stimulus intensity. Pain, 152(3), 592–598.

Breimhorst, M., Hondronikos, A. P., Elser, F., & Birklein, F. (2011). Cortical correlates of the autonomic effects of noxious stimulation in healthy man: An EEG study. The Journal of Pain, 12(1), 61–71.

Boucsein, W., Fowles, D. C., Grimnes, S., Ben-Shakhar, G., Roth, W. T., Dawson, M. E., & Filion, D. L. (2012). Publication recommendations for electrodermal measurements. Psychophysiology, 49(8), 1017–1034.

Dawson, M. E., Schell, A. M., & Filion, D. L. (2007). The electrodermal system. In J. T. Cacioppo, L. G. Tassinary, & G. G. Berntson (Eds.), Handbook of psychophysiology (3rd ed., pp. 159–181). Cambridge University Press.

Braithwaite, J. J., Watson, D. G., Jones, R., & Rowe, M. (2013). A guide for analysing electrodermal activity (EDA) and skin conductance responses (SCRs) for psychological experiments. University of Birmingham.
