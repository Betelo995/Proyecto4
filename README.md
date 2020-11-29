# Modelos Probabilísticos IE0405
# Proyecto 4
## Isaac Rojas Hernández
### B76693
### Viernes Lunes 30 de Noviembre del 2020
### Profesor Fabián Abarca

#### Parte 4.1
Para la primera parte se debía modificar el archivo para tener un modulador que en vez de utilizar un estándar BPSK a un QPSK. Esto se logra utilzando dos portadores ortogonales (en este caso sin y cos) en vez de únicamente uno. La forma de la onda que transportaría la imágen es: $s(t) = A_{1}cos(2\pi f_{c}t) + A_{2}sin(2\pi f_{c}t)$. Para poder lograr esto únicamente debemos modificar el modulador. 
A la hora de asignar bits a cada uno de los portadores debemos hacer 2 iteraciones en vez de 1 y recorrer los bits iniciando por el $bit[0]$ hasta el len(bit), pero en el caso del seno saltando cada 2 bits, es decir únicamente los pares y para el cos los impares. La implementación se ve de forma: 
```Python

# 2. Construyendo un periodo de la señal portadora c(t)
    Tc = 1 / fc  # periodo [s]
    t_periodo = np.linspace(0, Tc, mpp)
    portadora1 = np.sin(2*np.pi*fc*t_periodo) #Portadora 1 
    portadora2 = np.cos(2*np.pi*fc*t_periodo) #Portadora 2

    # 3. Inicializar la señal modulada s(t)
    t_simulacion = np.linspace(0, N*Tc, N*mpp) 
    senal_Tx1 = np.zeros(t_simulacion.shape)
    senal_Tx2 = np.zeros(t_simulacion.shape)
    moduladora = np.zeros(t_simulacion.shape)  # señal de información
 
 #luego incia con el primer portador asignando
    for i in range(0, len(bits), 2):
        bit1 = bits[i]
        
        if bit1 == 1:
            senal_Tx1[i*mpp : (i+1)*mpp] = portadora1

        elif bit1 != 1:
            senal_Tx1[i*mpp : (i+1)*mpp] = (portadora1 * -1)
            
    for i in range(1,len(bits), 2):
        bit2 = bits[i]
        if bit2 == 1:
            senal_Tx2[i*mpp : (i+1)*mpp] = portadora2

        elif bit2 != 1:
            senal_Tx2[i*mpp : (i+1)*mpp] = (portadora2 * -1)
```

Luego como tenemos las señales por separado es necesario unirlas, así como hacer una portadora universal que sea la suma de ambas, de forma que podamos utilizar la función de demodulación original, esto se vería de la siguiente forma: 
```Python
    senal_Tx =  senal_Tx1 + senal_Tx2 #se crea la señal conjunta
    portadora = portadora1 + portadora2 # Luego se crea la portadora conjunta
``` 

Una vez con estos dos cambios es posible utilizar el resto de las funciones de forma similar al ejemplo inicial, inclusive el demodulador, dando un resultado prácticamente impecable de transmisión de datos, aun con el ruido artificial creado. 


#### Parte 4.2
Para la parte 4.2 a la señal de transmisión se le debía aplicar las pruebas de estacionaridad y ergocicidad. Para ello se aplicaron los conocimientos adquiridos en el pasado Laboratorio 4. 
Inicialmente se creo una simulación de la variable 'senal_Tx', entonces inicialmente se obtenían los valores de cada función de forma que pudiesemos obtener la señal y todas sus posibles 4 combinaciones. Esto se logra con el código mostrado a continuación: 
```Python
for j in A: 
    s_1 = j*(np.sin(2*np.pi*fc*t) + np.cos(2*np.pi*fc*t))
    s_2 = j*(np.sin(2*np.pi*fc*t) - np.cos(2*np.pi*fc*t))
    s_t[j,:] = s_1
    s_t[j+1,:] = s_2
    plt.plot(t, s_1)
    plt.plot(t, s_2)

```

Una vez creados los valores de cada función se obtiene el promedio y el valores esperado para poder comparar ambos: 
```Python
#Ahora se obtienen los promedios
P = [np.mean(s_t[:,i]) for i in range(len(t))]
plt.plot(t, P, lw=6, color='y', label="Promedio de las realizaciones")

#Ahora obteniendo el valor esperado de la señal
E = np.mean(senal_Tx)*t 
plt.plot(t, E, '--' ,lw=2,color='b',label="Promedio teórico" )
```
Una vez obtenidos ambos comparamos en la gráfica generada, para obtener un resultado basado en la teoría de Estacionaridad y ergodicidad. Se puede apreciar de forma clara, que ambos resultados, tanto el teórico como el obtenido del experimento son iguales, ambos promedios son constantes con el tiempo, aún con la variación que tiene nuetras señal $S(t)$. Recordemos que para que haya estacionaridad, el promedio debe ser el mismo independientemente del tiempo en el que se tome. Además se dice que un proceso es ergódico cuando los promedios temporales igualan a los estadísticos, que de igual forma 'senal_Tx' cumple según las pruebas sometidas. 

#### Parte 4.3

Por último debíamos obtener la densidad espectral de potencia. Para poder obtener este resultado se debía importar la transformada de fourier del módulo de scipy. Luego se debía transformar 'senal_Tx' con esta transformada de fourier

```Python
senal_f = fft(senal_Tx)
```
Una vez transformada se sacaba las muetras de la señal, el número de símbolos, el tiempo del simbolo, tiempo entre muestras y tiempo de simulación. 
```Python
#Muestras de la señal
Nm = len(senal_Tx)

#Número de símbolos (198 x 89 x 8 x 3
Ns = Nm//mpp

#Tiempo del símbolo = periódo de la onda portadora
Tc = 1 / fc

#Tiempo entre muestras (periódo de muestreo)
Tm = Tc / mpp

#Tiempo de la simulación 
T = Ns * Tc

#Espacio de frecuencias
f = np.linspace(0.0, 1.0/(2.0*Tm), Nm//2)
```
Finalmente se graficaba con el espacio de frecuencia la fórmula dada para obtener el resultado deseado. 

