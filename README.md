# INTELIGENCIA ARTIFICIAL - UNIVERSIDAD DEL NORTE
PROGRAMA DE INGENIERÍA DE SISTEMAS <br>
PROFESOR: EDUARDO ZUREK, PH.D.<br>
<b>INTEGRANTES</b>:
<br>
<ul>
    <li>Johan Sebastian Burgos Guerrero</li>
    <li>Sebastian David Ariza Coll</li>
    <li>Daniel Valencia Cordero</li>
</ul>

# Tema: Juego de Damas con Poda Alfa-Beta

## Heuristica Utilizada
Para el desarrollo de la heuristica se tuvieron en consideración los siguientes aspectos [[1]](https://github.com/kevingregor/Checkers) dependiendo del jugador en turno:

1.   Cantidad de peones

2.   Cantidad de reinas

3.   Cantidad de fichas en la primera y última fila del tablero

4.   Cantidad de fichas en en la mitad del tablero, es decir, sea f el número de la fila y c el número de la columna. se consideran solo los valores para: (3<=f<=4 y 2<=c<=5) (verde)

5.   Cantidad de fichas en el el exterior del cuadro del centro (azul)

6.   Si una ficha puede ser eliminada, es decir si mi ficha es vulnerable

7.   Fichas protegidas

##  Calculo de los bordes del tablero
Se calcula las posiciones de los bordes para tener en cuenta en los movimientos de las fichas.

```
def Border(a):
  vecborder = [0,8,16,24,32,40,48,56,7,15,23,31,39,47,55,63,1,2,3,4,5,6,57,58,59,60,61,62]
  if (vecborder.count(a) == 0):
    return -1
  else:
    return 0
```

## #Movimiento de los peones

1. La a es la pos actual
2. La c es la dirreccion del movimiento, arriba o abajo, 1 o -1 respectivamente
3. La d es si me voy para la diagonal de la der o de la izq, 1 o -1 respectivamente

```
def MovPeon(tablero,a,c,d):
  pos = []
  nuvPos = a+(-1*c)*8+(-1*d)#esto es mover 1 cuadro en diagonal
  borIzq = [0,8,16,24,32,40,48,56]
  borDer = [7,15,23,31,39,47,55,63]  
  #print(a,c,d,nuvPos)
  if (nuvPos < len(tablero)):
    if (tablero[nuvPos] != "_"):#Valido que el nuvPos este vacio
      killpeonn = KillPeon(tablero,a,c,d)   
      #print("mata",a)     
      if (killpeonn[0] == "True"):
        pos.append("Kill")
        if (d== -1):
          pos.append("Izq")
        else:
          pos.append("Der")
        pos.append(killpeonn[1])       
      else:
        pos.append("Mov")
        pos.append("False")
    else:
      if ( (borDer.count(nuvPos) == 1 and d == 1) or (borIzq.count(nuvPos) == 1 and d == -1) ):#Valido que no me vaya pa' un borde
        pos.append("Mov")
        pos.append("False")  
      else:
        pos.append("Mov")
        pos.append(nuvPos)
  else:
    pos.append("Mov")
    pos.append("False") 
  return pos
```

## Heuristica Damas
Se creo una heuristica la cual obtiene todas las fichas que existen diagonalmente en un movimiento posible de la dama, se realiza una ponderacion de estas fichas tal que 
```
numY+2*numX - (numN+2*numO) 
```
y luego se retorna este valor para decidir el mejor movimiento de la dama.


## Diagonales
Se obtienes todas las posibles diagonales de una dama para revisar las fichas que existen a su lado.

## Movimiento de las damas

1. La a es la pos actual
2. La c es la dirreccion del movimiento, arriba o abajo, 1 o -1 respectivamente
3. La d es si me voy para la diagonal de la izq o de la dercha, -1 o 1 respectivamente
4. Se maneja de la misma manera de los peones solo que se analiza toda la diagonal y la heuristica decide cual es la mejor jugada para ambos jugadores.

```
def MovDama(tablero,minmax,a,c,d):
  pos = []
  heuMax = -10000
  vecbor = [0,8,16,24,32,40,48,56,7,15,23,31,39,47,55,63,1,2,3,4,5,6,57,58,59,60,61,62]
  nuvPos = a+(-1*c)*8+(-1*d)#esto es mover 1 cuadro en diagonal
  actuDama = []
  if (len(tablero) > nuvPos and nuvPos > 0):
    killDamaa, actuDama = killDama(tablero, minmax, a, c, d)
    #print(killDamaa, actuDama)
    if (actuDama[0] == "True"):
      pos.clear()
      pos.append("Kill")
      pos.append(killDamaa)
      pos.append(actuDama[1])
    else:
      while (tablero[nuvPos] == "_"):      
        if (len(tablero) > nuvPos and nuvPos > 0):
          if (vecbor.count(nuvPos) == 0):
            tem = HeuristicaDamas(tablero, nuvPos, minmax)
            if (tem > heuMax ):
              pos.clear()
              pos.append("True")
              pos.append(nuvPos)
              heuMax = tem 
          else:
            pos.append("False")
            pos.append(0)
            break 
        else:
          pos.append("False")
          pos.append(0)
          break
        nuvPos = nuvPos +(-1*c)*8+(-1*d)#esto es mover 1 cuadro en diagonal
      pos.append("False")
      pos.append(0)      
  else:
      pos.append("False")
      pos.append(0)  
  return pos
```
## Generar Hijos MinMax

1. La a es la pos actual
2. La c es la dirreccion del movimiento, arriba o abajo, 1 o -1 respectivamente
3. La d es si me voy para la diagonal de la der o de la izq, 1 o -1 respectivamente
4. Se tiene en cuenta las funciones de movpeon y movdama para analizar los posibles movimientos tanto a la derecha como a a la izquierda y con el vector decision que retorna cada uno de estas funciones se realiza el movimiento correspondiente.

## Algoritmo Poda Alfa Beta
Explicarlo: Se utilizo el algoritmo poda alfa y beta de forma normal y se analiza los resultados de cuando algun tablero existe un empate o un ganador.

```
def poda_alfa_beta(padre,alfa,beta,nivel,minmax,heuristica,sw):
    #basarnos en el del profe
    print("",end=nivel, file=f)
    if (minmax == "max"):
        print("Nodo max",file=f)
    else:
        print("Nodo min",file=f)
    print("",end=nivel, file=f)
    print(padre[0:8],file=f)
    print("",end=nivel, file=f)
    print(padre[8:16],file=f)
    print("",end=nivel, file=f)
    print(padre[16:24],file=f)    
    print("",end=nivel, file=f)
    print(padre[24:32],file=f)    
    print("",end=nivel, file=f)
    print(padre[32:40],file=f)    
    print("",end=nivel, file=f)
    print(padre[40:48],file=f)    
    print("",end=nivel, file=f)
    print(padre[48:56],file=f)    
    print("",end=nivel, file=f)
    print(padre[56:64],file=f)    
    print("",end=nivel, file=f)
    print(alfa,beta,file=f)
    alfanew = copy.deepcopy(alfa)
    betanew = copy.deepcopy(beta)    
    if (alfanew < heuristica)&(minmax == "max")&(abs(heuristica)>0):
        alfanew = copy.deepcopy(heuristica)
    if (betanew > heuristica)&(minmax == "min")&(abs(heuristica)>0):
        betanew = copy.deepcopy(heuristica)    
    #Esto se cambia para las Damas
    if (heuristica == 1000): # nodo hoja
        print("",end=nivel, file=f)      
        print("Gana X and Y",file=f)
    elif (heuristica == 100): # nodo hoja
        print("",end=nivel, file=f)      
        print("Empate",file=f)
    elif (heuristica == -1000): # nodo hoja
        print("",end=nivel, file=f)      
        print("Gana O and N",file=f)
    else: # se deben generar hijos
        nivelnew= copy.deepcopy(nivel)+"   |"
        print("",end=nivel, file=f)
        print("Generando hijos",file=f)
        hijos = generar_hijos(padre,minmax)
        #print(hijos)
        heuristicas = []
        for hijo in hijos:
            h1 = HeuristicaJuego(hijo,minmax)
            heuristicas.append(h1)
        hijos_ordenados = list(zip([abs(h1) for h1 in heuristicas],heuristicas,hijos))
        # ajustar si es nodo max o min
        hijos_ordenados.sort(reverse=True)
        podar = 0
        if (sw < 5):
          for orden,heur,hijo in hijos_ordenados:
            if (podar == 0):
                if minmax == "max":
                    alfat, betat = poda_alfa_beta(hijo,alfanew,betanew,nivelnew,"min",heur,sw+1)
                    if (alfanew < betat):
                        alfanew = copy.deepcopy(betat)
                        print("",end=nivel, file=f)
                        print(alfanew,betanew,file=f)
                else:
                    alfat, betat = poda_alfa_beta(hijo,alfanew,betanew,nivelnew,"max",heur,sw+1)
                    if (alfat < betanew):
                        betanew = copy.deepcopy(alfat)
                        print("",end=nivel, file=f)
                        print(alfanew,betanew,file=f)
                if (alfanew >= betanew):
                    print("",end=nivel, file=f)
                    print("Poda",file=f)
                    podar = 1
    return alfanew,betanew
```

