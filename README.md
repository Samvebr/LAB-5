# HRV Usando la Transformada Wavelet
## Fundamentos Teóricos
### Actividad Simpática y Parasimpática del Sistema Nervioso Autónomo 
**SNA Simpático** 

Es una parte importante del cuerpo durante situaciones de emergencias, lo que prepara al cuerpo durante las situaciones de emergencia, la llamada respuesta de “lucha o huida”.  El SN simpático activa diversas rutas donde se aumenta la frecuencia cardiaca y respiratoria, se eleva la presión sanguínea, dilatación de pupilas y cambios en el flujo sanguíneo para dirigir todo el volumen al cerebro, corazón y músculos. 

**SNA Parasimpático**

Ayuda a equilibrar el sistema nervioso simpático y controla la respuesta natural de relajación, después de haber estado en respuesta “lucha o huida”. Equilibra el cuerpo llevando a un estado de relajación mediante la disminución de la frecuencia cardiaca y la presión arterial.

### Efecto de la Actividad Simpática y Parasimpática en la Frecuencia Cardiaca
El sistema simpático aumenta la fuerza de contracción del corazón, aumenta la presión arterial, aumenta el gasto cardiaco, tensa los músculos y prepara los organismos para la acción inmediata. Por otro lado, el sistema parasimpático disminuye la frecuencia cardiaca, la presión arterial y el gasto cardiaco, calma la respiración favoreciendo la irrigación coronaria. 

<div align="center">
  <img src="https://github.com/user-attachments/assets/c38e24b5-509b-41c2-b61c-49f682dd9a13" width="80%">
</div>

### Variabilidad de la Frecuencia Cardiaca (HRV)
Las variaciones de la frecuencia cardiaca en los intervalos R-R reflejan la actividad del sistema nervioso autónomo y su capacidad de adaptabilidad a estimulos internos y externos. Una HRV elevada indica que la persona es saludable, por otro lado un HRV reducido puede asociarse con estrés, fatiga o enfermedades cardiovasculares.

El HRV se puede analizar en el dominio del tiempo o la frecuencia destancado bandas como la LF (0.04-0.15 Hz) la cual esta asociada con la actividad simpática o parasimpática, y la HF (0.15 - 0.4 Hz) que representa la modulación vagal y la respiración.
### Transformada Wavelet
Las Wavelet son señales, o formas de onda, las cuales tienen una duración limitada y un valor promedio de cero, pueden ser irregulares y asimétricas, esta característica les da una mejor comparación y análisis en contraste con la transformada de Fourier. En una señal continua los parámetros de escalamiento y desplazamiento dan paso a la obtención de los coeficientes wavelet. Los coeficientes nos indican cuanta relación hay entre la señal y la Wavelet madre. 
Algunos usos y tipos de Wavelet utilizados en señales biologicas son:

- **Electrocardiogramas (ECG)**: Detecta las ondas QRS y ayuda a la eliminación de ruido muscular y de artefacto. El tipo de wevelet utilizado son los *Daubechies (db)*; poseen buena localizacion en tiempo y frecuencia, *symlets(sym)*; tiene mayor simetria a comparación de los Daubechies y *Mexican Hat(Ricker)*; utiliza la segunda derivada de la gaussiana.
- **Electroencefalogramas (EEG)**: Con el fin de detectar picos epilépticos y análisis de fases del sueño. Se emplean los *Coiflets(coif)*; que poseen alta simetria y mayor cantidad de momentos nulos, lo que permite la multiresolución y *Morlet*; de manera sinusoidal modula la frecuencia.
- **Fonocardiogramas(PCG)**: Identificación de ruidos cardiacos anormales y soplos. Los tipos de Wavelet empleados para el analisis de esta señal biológica son *Symlets(sym)*; buena locaclización en tiempo y frecuencia con mayor  simetria y *Biorthigonal*; empleada en reconstruccion y simetria, ayuda en la compresión de imágenes médicas. 
##Procedimientos

### Diagrama de flujo
- Mediante este se mostrara el plan de acción y su paso a paso con el fin de obtener un experimento con la organización y rigurosidad necesaria.
  
![Image](https://github.com/user-attachments/assets/e6d49e42-c604-4486-b6ab-e33da73f28fe)

### Adquisición de la Señal ECG
Para la adquisición de la señal se tomo al sujeto de prueba, posteriormente se le colaron los electrodos y mediante comunicacion serial se obtubieron los datos de la frecuencia cardiaca del sujeto en reposo y en actividad durante 5 minutos, que posteriormente son exportados y guardados en un archivo *csv*. Los datos se obtuvieron con Matlab.

Primero se cerraron los puertos eriales abiertos para evitar conflictos con aperturas posteriores, la instrucción *serialportfind* detecta los puertos abierto y con *delete* se cierran.

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
Luego se configura el el puerto serial para establecer la conexion con la STM32, con *puerto* se especifica a donde esta conectado el dispositivo y con *baudRate* la velocidad de la comunicacion serial.

``` bash
%% Configuración del puerto serial
puerto   = "COM4";   % Ajusta según corresponda
baudRate = 9600;
s = serialport(puerto, baudRate);
configureTerminator(s, "LF");
```
Se configuran los parametros realacionados al ADC y la visualizacion, como lo es la resolucion, la frecuencia de muestreo, el intervalo de tiempo entre muestras y el numero de muestras. Tambien se inicializa el tiempo de ventana para graficar los vectores y almanecer los datos.

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
Se crea la ventana grafica y la interfaz de usuario para poder observar la toma de la señal, además se configura un boton de *stop* para poder detener la toma de la señal de ser necesario, tambien se configuran los ejes y las leyendas del grafico.

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
Se crea un bucle para adquirir los datos de forma infinita hasta ser presionado *stop*, luego los datos se convierten a voltaje para ser graficados en tiempo real, cada muestra de estos datos se almacena en *dataLog* y su tiempo en *timeLog*.

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
Por ultimo se guardan los datos en un archivo CSV y se cierra la comunicacion serial y la interfaz gráfica.

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
- Lo que haga Salo
### Analísis del HRV
- El HRV de cada individuo es diferente y este depende a condiciones de edad y de salud, el caso de analísis es sobre Samuel Velandia, joven deportista de 20 años.
#### Procesamiento de la señal
- El codigo con el cual se analizara el paciente esta construido de la siguiente forma:
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
- 
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
- 
#### Minuto 3:

![Image](https://github.com/user-attachments/assets/e613f1c5-830f-42bf-b201-c23d167fac31)}

- Media R-R: 0.8438 s
- Desviación estándar R-R: 0.0642 s

#### Minuto 4:

![Image](https://github.com/user-attachments/assets/d91e317a-8f2a-4c39-96d1-8b8bb49cdcfa)

- Media R-R: 0.9051 s
- Desviación estándar R-R: 0.1282 s

### Aplicación transformada Wavelet


