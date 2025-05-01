# HRV Usando la Transformada Wavelet
## Fundamentos Teóricos
### Actividad Simpática y Parasimpática del Sistema Nervioso Autónomo 
**SNA Simpático** 

Es un mecanismo o parte del sistema nervioso que permite a nuestro cuerpo tener una respuesta optima ante una situacion de emrgencia, tambien conocido como la respuesta de “lucha o huida”.  El SN simpático activa diversas rutas donde se aumenta la frecuencia cardiaca y respiratoria, se eleva la presión sanguínea, dilatación de pupilas y cambios en el flujo sanguíneo, esto para dirigir todo el volumen al cerebro, corazón y músculos. 

**SNA Parasimpático**

Ayuda a equilibrar el sistema nervioso simpático y controla la respuesta natural de relajación, después de haber estado en respuesta “lucha o huida”. Equilibra el cuerpo llevandolo a un estado de relajación mediante la disminución de la frecuencia cardiaca y la presión arterial.

### Efecto de la Actividad Simpática y Parasimpática en la Frecuencia Cardiaca
El sistema simpático aumenta la fuerza de contracción del corazón, aumenta la presión arterial, aumenta el gasto cardiaco, tensa los músculos y prepara los organismos para la acción inmediata. Por otro lado, el sistema parasimpático disminuye la frecuencia cardiaca, la presión arterial y el gasto cardiaco, este tambien calma la respiración favoreciendo la irrigación coronaria. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/c38e24b5-509b-41c2-b61c-49f682dd9a13" width="80%">
</div>

### Variabilidad de la Frecuencia Cardiaca (HRV)
Las variaciones de la frecuencia cardiaca en los intervalos R-R reflejan la actividad del sistema nervioso autónomo y su capacidad de adaptabilidad a estimulos internos y externos. Una HRV elevada indica que la persona es saludable, por otro lado un HRV reducido puede asociarse con estrés, fatiga o enfermedades cardiovasculares.

El HRV se puede analizar en el dominio del tiempo o la frecuencia destacando bandas como la LF (banda de frecuencia baja) (0.04-0.15 Hz) la cual esta asociada con la actividad simpática o parasimpática, y la HF (banda de frecuencia alta) (0.15 - 0.4 Hz) que representa la modulación vagal y la respiración.
### Transformada Wavelet
Las Wavelet son señales, o formas de onda, las cuales tienen una duración limitada y un valor promedio de cero, pueden ser irregulares y asimétricas, esta característica les da una mejor comparación y análisis en contraste con la transformada de Fourier. En una señal continua los parámetros de escalamiento y desplazamiento dan paso a la obtención de los coeficientes wavelet. Los coeficientes nos indican cuanta relación hay entre la señal y la Wavelet madre. 
Algunos usos y tipos de Wavelet utilizados en señales biologicas son:

- **Electrocardiogramas (ECG)**: Detecta las ondas QRS y ayuda a la eliminación de ruido muscular y de artefacto. El tipo de wevelet utilizado son los *Daubechies (db)*; poseen buena localizacion en tiempo y frecuencia, *symlets(sym)*; tiene mayor simetria a comparación de los Daubechies, *Mexican Hat(Ricker)*; utiliza la segunda derivada de la gaussiana y *Morlet*; de manera sinusoidal modula la frecuencia..
- **Electroencefalogramas (EEG)**: Con el fin de detectar picos epilépticos y análisis de fases del sueño. Se emplean los *Coiflets(coif)*; que poseen alta simetria y mayor cantidad de momentos nulos, lo que permite la multiresolución y *Morlet*; de manera sinusoidal modula la frecuencia.
- **Fonocardiogramas(PCG)**: Identificación de ruidos cardiacos anormales y soplos. Los tipos de Wavelet empleados para el analisis de esta señal biológica son *Symlets(sym)*; buena locaclización en tiempo y frecuencia con mayor  simetria y *Biorthigonal*; empleada en reconstruccion y simetria, ayuda en la compresión de imágenes médicas.

### Diagrama de flujo
- Mediante este se mostrara el plan de acción y su paso a paso con el fin de obtener un experimento con la organización y rigurosidad necesaria.
  
![Image](https://github.com/user-attachments/assets/e6d49e42-c604-4486-b6ab-e33da73f28fe)

![Imagen de WhatsApp 2025-04-30 a las 10 31 57_de3500cf](https://github.com/user-attachments/assets/d1c5c989-6577-41f2-a7b7-e5611d3059b8)

      

![Imagen de WhatsApp 2025-04-30 a las 10 48 02_f2c4e05f](https://github.com/user-attachments/assets/9fb62097-4b40-4bc5-9972-1aa0e6416d3c)




### Adquisición de la Señal ECG
Para la adquisición de la señal se tomo al sujeto de prueba, posteriormente se le colaron los electrodos y mediante comunicacion serial se obtuvieron los datos de la frecuencia cardiaca del sujeto en reposo y en actividad durante 5 minutos, que posteriormente son exportados y guardados en un archivo *csv*. Los datos se obtuvieron mediante una interfaz en Matlab.

- Primero se cerraron los puertos eriales abiertos para evitar conflictos con aperturas posteriores, la instrucción *serialportfind* detecta los puertos abierto y con *delete* se cierran.

``` bash
clc;
clearvars;
close all;

%% Cerrar puertos seriales abiertos previamente
puertosAbiertos = serialportfind;
if ~isempty(puertosAbiertos)
    for idx = 1:length(puertosAbiertos)
        delete(puertosAbiertos(idx));
    end
end
```
- Luego se configura el el puerto serial para establecer la conexion con la STM32, con *puerto* se especifica a donde esta conectado el dispositivo y con *baudRate* la velocidad de la comunicacion serial.

``` bash
%% Configuración del puerto serial
puerto   = "COM4";   % Ajusta según corresponda
baudRate = 9600;
s = serialport(puerto, baudRate);
configureTerminator(s, "LF");
```
- Se configuran los parametros realacionados al ADC y la visualizacion, como lo es la resolucion, la frecuencia de muestreo, el intervalo de tiempo entre muestras y el numero de muestras. Tambien se inicializa el tiempo de ventana para graficar los vectores y almanecer los datos.

``` bash
voltaje_ref  = 5;        % Voltaje de referencia del ADC (V)
adc_max      = 255;      % Resolución 8 bits
fs           = 2000;     % Frecuencia de muestreo estimada (Hz)
dt           = 1/fs;     % Intervalo de muestreo (s)
num_muestras = 3500;     % Puntos en la ventana de pantalla
offset       = 0.0;      % Offset (V)

tiempoVentana = (0:num_muestras-1) * dt;
buffer        = zeros(1, num_muestras);

dataLog = [];
timeLog = [];
```
- Se crea la ventana grafica y la interfaz de usuario para poder observar la toma de la señal, además se configura un boton de *stop* para poder detener la toma de la señal de ser necesario, tambien se configuran los ejes y las leyendas del grafico.

``` bash
gcfHandle = figure('Name','EMG en Tiempo Real','NumberTitle','off');
set(gcfHandle, 'UserData', struct('stopFlag', false)); % Inicializar UserData
set(gcfHandle, 'CloseRequestFcn', @(src,~) figureClose(src));

uicontrol('Style','pushbutton','String','Stop','Position',[10 10 50 20],...
    'Callback',@(src,~) stopAcquisition(src));

hLine = plot(tiempoVentana, buffer, 'b', 'LineWidth', 1.5);
ylim([0, voltaje_ref + offset]);
xlim([tiempoVentana(1), tiempoVentana(end)]);
xlabel('Tiempo (s)');
ylabel('Voltaje (V)');
title('Señal EMG en Tiempo Real');
grid on;
```
- Se crea un bucle para adquirir los datos de forma infinita hasta ser presionado *stop*, luego los datos se convierten a voltaje para ser graficados en tiempo real, cada muestra de estos datos se almacena en *dataLog* y su tiempo en *timeLog*.

``` bash
userData = get(gcfHandle, 'UserData');
while ~userData.stopFlag && ishandle(gcfHandle)
    if s.NumBytesAvailable > 0
        nuevosBytes = read(s, s.NumBytesAvailable, 'uint8');
        nuevosVolt  = double(nuevosBytes)/adc_max * voltaje_ref + offset;
        for v = nuevosVolt(:).'
            buffer = [buffer(2:end), v];
            dataLog(end+1) = v;
            timeLog(end+1) = (length(dataLog)-1) * dt;
        end
        set(hLine, 'YData', buffer);
        drawnow limitrate;
    else
        pause(0.001);
    end
    userData = get(gcfHandle, 'UserData'); % Actualizar UserData
end
```
- Por ultimo se guardan los datos en un archivo CSV y se cierra la comunicacion serial y la interfaz gráfica.

``` bash
if ~isempty(dataLog)
    filename = fullfile(pwd, 'Laboratorio_Corazon.csv');
    T = table(timeLog.', dataLog.', 'VariableNames', {'Tiempo_s','Voltaje_V'});
    writetable(T, filename);
    disp(['Datos guardados en: ', filename]);
end
delete(s);
if ishandle(gcfHandle)
    delete(gcfHandle);
end

%% Callbacks
function stopAcquisition(src)
    userData = get(src.Parent, 'UserData');
    userData.stopFlag = true;
    set(src.Parent, 'UserData', userData);
end

function figureClose(src)
    userData = get(src, 'UserData');
    userData.stopFlag = true;
    set(src, 'UserData', userData);
    delete(src);
end
```

**Interfaz Grafica de Matlab**

![Imagen de WhatsApp 2025-04-23 a las 11 46 14_ecec06fc](https://github.com/user-attachments/assets/3db827cc-d956-403a-881d-43fa9e491a58)

### Diseño e implementación filtro IIR

Un filtro IIR (Infinite Impulse Response) es un filtro digital que funciona en base a los valores presentes de la señal y sus valores pasados, muy utilizados por su alta eficiencia y su uso cuando contamos con recursos limitados (embebidos). Por tanto se diseño un filtro pasa banda de orden 4, con el objetivo de permitir el paso de señales en el rango 0,5 Hz a 40 Hz, eliminando componentes de baja y alta frecuencia.
Los parametros empleados para el filtro fueron:
- fs= 200
- lowcut= 0,5 Hz
- highcut= 40 Hz
- orden = 4
- frecuencia Nyquist = fs/2 = 100
- Frecuencia normalizada
    - low = 0,5/100 = 0,005
    - high = 40/100 = 0,4
Se eligio este tipo de filtro puesto que tiene una respuesta de magnituf suave y sin ondulaciones en la banda pasante, ademas de poseer una respuesta transitoria estable y es adecuado para preservar la forma de las señales fisiológicas.

La ecuación en diferencias esta expresada por:

![image](https://github.com/user-attachments/assets/cb1c8853-7930-47f1-9a21-37008f335105)

Donde los coeficientes b<sup>k</sup> son los coeficeintes del numerador y a<sup>k</sup> los coeficientes del denomidor.Los coeficientes se encontraron con ayuda del siguiente codigo.
``` bash
 b, a = butter(4, [low, high], btype='band')
```
Diseña el filtro con polos complejos conjugados en el plano *S*, luego aplica la transfromada bilineal para convertir el filtro analogo *H(s)*  a un filtro digital *H(z)* y retorna los coeficientes a y b de la funcion de transferencia en *H(z)*.

$$
H(z) = \frac{b_0 + b_1 z^{-1} + \dots + b_M z^{-M}}{a_0 + a_1 z^{-1} + \dots + a_N z^{-N}}
$$

Con los coeficientes obtenidos la ecuacion diferencial quedaria tal que asi:

$$
\begin{aligned}
y[n] &=\ 0.0448 \cdot x[n] - 0.1791 \cdot x[n-2] + 0.2686 \cdot x[n-4] \\
     &\quad - 0.1791 \cdot x[n-6] + 0.0448 \cdot x[n-8] \\
     &\quad - 4.7666 \cdot y[n-1] + 9.7784 \cdot y[n-2] - 11.6066 \cdot y[n-3] \\
     &\quad + 9.0377 \cdot y[n-4] - 4.7893 \cdot y[n-5] + 1.6311 \cdot y[n-6] \\
     &\quad - 0.3161 \cdot y[n-7] + 0.0315 \cdot y[n-8]
\end{aligned}
$$

#### Codigo del filtro
``` bash
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
from scipy.signal import butter, lfilter, find_peaks
```
- Las librerias escenciales para procesar la señal y la función de la libreria scipy que importa los filtros.
``` bash

def diseño_filtro_IIR(fs, lowcut, highcut, order=4):
    nyq = 0.5 * fs
    low = lowcut / nyq
    high = highcut / nyq
    b, a = butter(order, [low, high], btype='band')
    return b, a
```

- Primero diseñamos el filtro, teniendo en cuenta las frecuencias de corte y el orden, este ultimo esta dado por la eficiencia con la que se va a filtrar la señal, tambien se configuran las frecuencias de muestreo por el teorema de Nyquist.

``` bash

def aplicar_filtro(data, b, a):
    return lfilter(b, a, data)
``` 
- Aplicamos el filtro creando una función, esta tomara los datos guardados en la variable data y le aplicara los valores a y b del filtro, retornando la función filtrada


### Analísis del HRV
- El HRV de cada individuo es diferente y este depende a condiciones de edad y de salud, el caso de analísis es sobre Samuel Velandia, joven deportista de 20 años.
#### Procesamiento de la señal
El codigo con el cual se analizara el paciente esta construido de la siguiente forma:
 ``` bash
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy.signal import butter, lfilter, find_peaks
```
- Estas librerias son necesarias para la lectura operación matematica y graficación de los datos adquiridos y su filtrado.

-Se declaran 5 funciones escenciales

``` bash
def aplicar_filtro(data, b, a):
    return lfilter(b, a, data)
```
- Aplicación del filtro a la señal.

``` bash
def detectar_picos_R(signal, fs):
    peaks, _ = find_peaks(signal, distance=fs*0.6, height=np.mean(signal))
    rr_intervals = np.diff(peaks) / fs
    return peaks, rr_intervals
```
-  Detección de los picos correspondientes al intervalo R-R, esto basado en los valores medios de la señal

``` bash
# 4. Generar nueva señal basada en RR intervalos
def señal_rr(rr_intervals, fs):
    freq_hr = 60 / rr_intervals
    rr_signal = np.zeros(int(np.sum(rr_intervals) * fs))
    pos = 0
    for interval, bpm in zip(rr_intervals, freq_hr):
        rr_signal[int(pos):int(pos + interval * fs)] = bpm
        pos += interval * fs
    return rr_signal
```
- Generamos una señal basada en los intervalos R-R, tomando los picos en base a su frecuencia.
 
``` bash 
def calcular_hrv(rr_intervals):
    if len(rr_intervals) == 0:
        return None, None
    media_rr = np.mean(rr_intervals)
    std_rr = np.std(rr_intervals)
    return media_rr, std_rr
```
- Finalmente tomamos los intervalos y calculamos su media en el dominio del tiempo

#### Calculo estadistico
- Utilizamos las funciones para calcular la desviación estandar y la media del siguiente modo:
  ``` bash
        media_rr, std_rr = calcular_hrv(rr_intervals)

        # Imprimir resultados de HRV
        print(f"Segmento {i+1} (minuto {i+1}):")
        if media_rr is not None:
            print(f"  - Media R-R: {media_rr:.4f} s")
            print(f"  - Desviación estándar R-R: {std_rr:.4f} s\n")
        else:
            print("  - No se detectaron suficientes picos R.\n")

  ```
### Graficas y analísis de datos
#### Minuto 1:

![Image](https://github.com/user-attachments/assets/9abd625e-e866-43be-b56b-a8bbc511e6dc)

- Media R-R: 0.8755 s
- Desviación estándar R-R: 0.0860 s
- En el primer minuto el sujeto esta en reposo y tiene una frecuencia cardiaca aproximada de 70 bpm, concordando la media de los latidos (Más o menos un latido cada segundo).
  
#### Minuto 2:

![Image](https://github.com/user-attachments/assets/6212df41-9c71-428d-b1f9-c656c09f7d0e)

- Media R-R: 0.8397 s
- Desviación estándar R-R: 0.0658 s
- En la mitad de este segundo instante el paciente se somete a apnea durante 60 segundos con el fin de que se evidencie un proceso parasimpatico y disminuya su frecuencia cardiaca.
  
#### Minuto 3:

![Image](https://github.com/user-attachments/assets/e613f1c5-830f-42bf-b201-c23d167fac31)}

- Media R-R: 0.8438 s
- Desviación estándar R-R: 0.0642 s
- La apnea anteriormente mencionada genero una bajada en el ritmo cardiaco, esto puede ser observado en la grafica relacionada en los intervalos R-R donde se ve como la frecuencia a los largo del tiempo tiende a tener valor entre 60-70 bpm.
  
#### Minuto 4:

![Image](https://github.com/user-attachments/assets/d91e317a-8f2a-4c39-96d1-8b8bb49cdcfa)

- Media R-R: 0.9051 s
- Desviación estándar R-R: 0.1282 s
- Finalmente en el ultimo instante la la frecuencia en un primer instante de 10 segundos es bastante mas veloz que en los siguientes 50 segunods, esto debido a la compensación que genera el corazon al volver a respirar y regular todas las actvidades fisiologicas.

### Aplicación transformada Wavelet a la señal

#### Codigo e implementación

``` bash
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy.signal import butter, lfilter, find_peaks
import pywt
```
- Importamos las mismas librerias que se utilizan para el analísis del HRV, a excepción de utilizamos ahora la libreria que se encarga del uso de la transformada Wavelet continua.

``` bash
def aplicar_wavelet(signal, fs, segmento_id):
    wavelet = 'cmor1.5-1.0'  # Complex Morlet
    total_time = len(signal) / fs
    time = np.linspace(0, total_time, len(signal))
```
- Primero se crea una función que se encarga de aplicar la transformada con tres parametros, fs, signal y segmento_id), acá asignamos los parametros como el tiempo total de la señal que se analiza, variamos un valor Morlet a la variable Wavelet esta en función de como vamos a ver percibir la grafica.

``` bash
    # Escalas relacionadas con frecuencia
    freqs = np.linspace(0.04, 0.4, num=100)  # HRV bands
    scales = pywt.central_frequency(wavelet) * fs / freqs
    coef, freqs = pywt.cwt(signal, scales, wavelet, sampling_period=1/fs)
    power = np.abs(coef) ** 2
```
- En este cuadro definimos variables como la frecuencia, la escala, coeficientes la frecuencia, y el calculo para obtener la potencia espectral.

### Graficas y analísis Wavelet
#### Espectro 1:

![Image](https://github.com/user-attachments/assets/18092cf0-5f35-40c2-bff3-ab50bb9d6872)

Analisis: Como se observa en la grafica la mayor parte de la potenica espectral se concentra en la banda que esta por debajo de 0.15 Hz (Lf), principalmente en los instantes iniciales y finales, teniendo en cuenta que en el primer minuto el paciente se encontraba experimentalmente en "estrés" podemos relacionarlo con el comportamiento del estrés agudo del sistema simpatico cuando se activa, lo que tiende a reducir la variabilidad rapida (Hf) y  a aumentar las fluctuaciones lentas (Lf) que estan asociadaas a control simpatico y parasimbatico combinados, en relacion con el comportamiento fisiologico un pico de potencia en Lf puede reflejar osiclaciones lentas en la frecuencia cardiaca, este comporatmiento es comun de la modulacion simpatica/alfa relacionada a los reflejos, lo cual en relacion con el estres experimental del primer minuto del paciente es coherente.
#### Espectro 2:

![Image](https://github.com/user-attachments/assets/a38f2f63-24db-4537-8205-259a4231551c)

Analisis: Durante el segundo minuto el paciente tendria que estar en relajacion, sin embargo, como se observa en el espectograma sigue habiendo una potencia en Lf, sin embargo no se ve de forma tan intensa, esto puede ser el resultado de relajara al paciente, ya que reaparece parte de la modulacion parasimpatica respiratorio rapida (HF), ligada a la respiracion, ya que el Hf refleja variaciones debidos al nervio vago y la respiracion, se observa en la imagen como su recuperacion indica que el sistema vagal vuelve a modular con mas fuerza la frecuencia cardiaca
#### Espectro 3:

![Image](https://github.com/user-attachments/assets/bdd4e09e-18c9-48c4-9790-56959e876d38)

Analisis: En el minuto numero 3 el paciente volvio a realizar un esfuerzo para lograr resultados de "estres" , podemos observar como se refuerza de nuevo la potencia en la banda LF y se atenua la HF, esto debido a que volvimos aplicar la fase de estres, es decir, vuelve la dominancia simpatica, desplazando la energia hacia oscilaciones mas lentas.
#### Espectro 4:

![Image](https://github.com/user-attachments/assets/1416cbd3-9ae4-40cc-8153-60cfc960db6b)

Analisis: En este segmento se puede apreciar una ligera señal en Hf junto a todavia algo de Lf residual, esto se debe a que aunque sea una fase de relajacion, pueden quedar "inercia simpatica" que ralentiza la recuperacion completa de Hf, aunque no se pueda observar detenidamente una buena fase de relajacion, es posible evaluar la rapidez de retorno vagal tras una fase de estrés.


**Explicacion de resultados en cuanto a Bandas LF y Hf:**
Aunque si es cierto que mayormente relacionamos el concepto de "mas lento" con una mayor relajacion y viceversa, en el contexto de HRV en realidad funciona de la siguiente manera:

Bandas Lf/Oscilaciones lentas: El estres activa el sistema simpatico, esto hace que se refuerze estas oscilaciones lentas, lo cual se refleja como un mayor potencial en Lf cuando un cuerpo responde a un reto, es decir, elevacion de preion, frecuencia cardiaca establecida

Bandas Hf / Oscilaciones Rapidas: Como se nombro en el marco teorico esta banda esta relacionada con la modulacion parasimpatica que a su ves esta ligada a la respiracion, por lo que en relajacion prasimaptica fuerte, la potencia Hf aumenta, y las oscilasciones del ritmo respiratorio son muy marcadas.

Conclusiones:

Oscilaciones de LF "lentas" en nuestro resultados reflejan la accion simpatica y la autorregulacion de la presion arterial, mas no un estado de paz, seria correcto decir que el paciente estaba en un estado de bajo estres, sin embargo no de calma, lo cual encaga con las caracteristicas del paciente, ya que al tomar la señal, el sujeto no presentaba las los rasgos o comportamientos de una persona serena o calmada.

Oscilaciones HF "Rapidas" son variaciones respiratorias y vagales, lo que nos indica un esfuerzo del sistema para llegar a un estado de recuperacion y calma.

Para obtener unos resultados en los cuales podamos analizar bien el comportamiento de la variabilidad cardiaca en situaciones de calma o estres, es recomendable tomar dos seañles por separado con el mismo intervalo del tiempo y cada uno con una duracion de almenos 5 minutos, ya que variar de un minuto a otro fases de "calma" y "estrés" da lugar a visualizar estados de "retorno vagal" como en el ultimo resultado de este esperimento.
  
