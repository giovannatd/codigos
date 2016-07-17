# codigos
Calculadora IMC

//IFPB - Campus João Pessoa
//Disciplica: MICROPROCESSADORES E MICROCONTROLADORES
//Professor: Ilton L. Barbacena
//Calculadora de IMC, utilizando teclado resistivo com leitor de tensão e 16 displays 7-seg conectados em cascata com max7219 no atmega 328
//Dupla: Felipe Suassuna Guedes  e  Giovanna Targino Dália
//Video disponível: https://youtu.be/aYdR3nqz35w


#include "LedControl.h"
//LedControl lc = LedControl(INPUT,CLK, LOAD, N);
LedControl lc = LedControl(13,11,12,2);

/*const int mapa[4][4]={          //VALORES REAL           
          {522,331,255,202},
          {600,360,272,213},
          {720,400,294,227},
          {875,445,317,240}
};
*/
const int mapa[4][4]={              // VALORES PARA SIMULAÇÃO
          {876,720,599,525},
          {441,398,358,330},
          {316,294,271,255},
          {240,227,213,203}
};

/*const char tecla[4][4] = {   // VALORES TECLADO REAL
          {'1','2','3','B'},
          {'4','5','6','*'},
          {'7','8','9','-'},
          {'.','0',13,'+'}
};*/

const char tecla[4][4] = { // VALORES TECLADO SIMULAÇÃO
          {'7','8','9','B'},
          {'4','5','6','*'},
          {'1','2','3','-'},
          {'.','0',13,'+'}
};

//variaveis para fazer o blink nos displays
unsigned long t1, t2; 
boolean bli;

/* 
/* 
    FORMATO DAS LETRAS UTILIZADAS EM BINARIO
    
 A    B1110111
 b    B0011111
 C    B1001110
 d    B0111101
 E    B1001111
 F    B1000111
 G    B1011111
 H    B0110111
 I    B0110000
 J    B1111100
 L    B0001110
 M    B1100110 + B1110010
 n    B0010101
 O    B1111110
 P    B1100111
 q    B1110011
 R    B1110111
 S    B1011011
 t    B0001111
 u    B0111110
 V    B0111110
 W   ---
 X ---
 Y---
 Z   = 2
  
 */

int v2 = A1; // pino de entrada do teclado no arduino
int avg, adc[10];
int median=0;

void setup() {
    //Serial.begin(9600);
    //Serial.println("comecar os trabalhos");
    
    for(int i =0; i<2; i++)//2 = total de max 7219 utilizado
    {
        lc.shutdown(i, false); 
        lc.setIntensity(i, 10);
        lc.clearDisplay(i);
    }
    
    //textos escritos nos displays
    //utilizando função LedControl.setRow(N° max7219,N° display, caracter em bi ou hexa)
    //{
    text1();
    limpaa();
    text2();
    limpaa();
    text3();
    limpaa();
    
}

void loop() {
      limpaa();
      text4();
      limpaa();
      text5();
      //}    
    
      float peso =0, altura =0, imc=0;

      //Serial.println("Digite seu peso em KG: ");

 
         //  peso = valores(0);
           //Serial.println(" ");
           while ((peso<1)||(peso > 400)) // so aceitando assim, pesos maiores que 0 ou menores que 401
           {
                for (int xi=0;xi <6;xi++)
                    lc.setRow(0,xi,B0000000); 
                //Serial.println();
                //Serial.println("Valor invalido, digite um valor maior que zero");
                //Serial.println();
                //Serial.println("Digite seu peso em KG:");
                peso = valores(0);        
                //Serial.println();                
                
            }
            
        

          //Serial.println("Digite sua altura em cm:");          
  
          altura = valores(1);
           while ((altura<1) || (altura > 250)) //aceitando somente alturas maiores que 0 e também menores que 251 
           {
                for (int xi=0;xi <6;xi++)
                    lc.setRow(1,xi,B0000000); 
                //Serial.println();
                //Serial.println("Valor invalido, digite um valor maior que zero"); 
                //Serial.println();
                //Serial.println("Digite sua altura em cm");
                altura = valores(1);
                //Serial.println();
            }
        
     
          //Serial.println(" ");

          float alturacalc = altura/100;
          float alturaf= alturacalc*alturacalc;

          imc= peso/alturaf; //dado que o calculo de imc = peso/altura^2

          imc = (imc);
          limpaa();
          //Serial.print("IMC = ");
          String simc = String(imc);  //passa o imc para string para escrever nos displays
          simc = simc.substring(0,simc.indexOf(".")+3); // para usar somente 2 casas decimais
          //Serial.println(simc);

          timc();
          int pt =0;          
          for(int lsk =0; lsk<simc.length(); lsk++){  //escrevendo o IMC nos displays
              if( simc[lsk] =='.'){
                   lc.setChar(1,8-lsk,simc[lsk-1],true);
                   pt++;
              }
              else
                   lc.setChar(1,7-lsk+pt,simc[lsk],false);                    
              
          }
          delay(3000); //tempo que o resultado do imc ficara disposto nos displays
              
          //Serial.println(" ");

          if (imc<17)
          {
              //Serial.println("==> Voce esta muito abaixo do peso <==");
              //Serial.println(" ");
              magreza_II();
              //lcd.print("Mt abaixo do peso");
              delay(3000);
          }
          if (imc>=17 && imc<=18.49)
          {
              //Serial.println("==> Voce esta abaixo do peso <==");
              //Serial.println(" ");
              magreza_I();
              //lcd.print("Abaixo do peso");
              delay(3000);
          }
          if(imc>=18.5 && imc<=24.99)
          {
              //Serial.println("==>Voce esta saudavel<==");
              //Serial.println(" ");
              normal();
             //lcd.print("Peso Normal");
              delay(3000);
          }
          if(imc>=25.0 && imc<=29.99)
          {
              //Serial.println("==>Peso em Excesso<==");
              //Serial.println(" ");
              acima_do_peso();
             // lcd.print("Acima do peso");
              delay(3000);
          }
          if(imc>=30 && imc<=34.99)
          {
              //Serial.println("==>Obesidade Grau 1<==");
              //Serial.println(" ");
              obesidade_I();
              //lcd.print("Obesidade 1");
              delay(3000);
          }
          if(imc>=35 && imc<=39.99)
          {
              //Serial.println("==>Obesidade Grau 2<==");
              //Serial.println(" ");
              obesidade_II();
              //lcd.print("Obesidade 2");
              delay(3000);
          }
          if(imc>=40)
          {
              //Serial.println("==>Obesidade Grau 3<==");
              //Serial.println(" ");
              obesidade_III();
             // lcd.print("Obesidade 3");
              delay(3000);
          } 
        

        
}



char ler_teclado(int kh, int zh) //2  com os valores analogicos de tensão essa função salva os valores em um vetor, atraves de mediana
//chama o InsertionSort para esse vetor e calcula mediana, para identificar a tecla sem possivel ruidos.
{
  
        int  cont;
        char r='M';



        cont = 0;
        ler_tecla(kh,zh,cont);     
        
        while(cont < 10 )        // le 10 vezes
        { 
                adc[cont]  = ler_tecla(kh, zh, cont);     // 0 ... 1023
                delay(19);
                cont++;

        } 
        InsertionSort(adc, 10);
        median = (adc[4] + adc[5])/2;

        r =  veja(median);
        return r;
}

char veja(int r1)           //3  faz comparação entre o mapa de tensões e ver no vetor tecla qual seria a correspondente,  retornando a tecla pressionada identificada
{

        int st;
        char m='w';

        
        st = 0;
        for(int i =0;i<4;i++)
        {
            for(int j = 0;j<4;j++)
            {
              if((r1 >= mapa[i][j]-5) && (r1 <= mapa[i][j]+5))    // mapeamento +-5
              {
                  m = tecla[i][j];   // tecla lida
                  st = 1;
                  break;
              } 
            }
            if(st)        // tecla ja lida
              break;
         }

         
         return (m);
}

int ler_tecla(int lh, int th, int cont) // 1    Ler o valor de tensão analogica (100.. 1023) para evitar ruidos 
//Blink 
{

        int ler=0;
        
        while(1)    // equanto ler =0  => nenhuma tecla pressionada
        { 
                ler = analogRead(v2);      // ler tensao analogica (ADC => 0 ... 1023)
                if (ler > 100)     // tensao > 0
                  break;

                t1 = millis();
                if(t1-t2>500){
                    t2 = t1;
                    if(bli == 0) 
                        lc.setRow(th,5-lh,B0000000);
                    else
                        lc.setRow(th,5-lh,B0001000);
                                           
                   bli = !bli; 
                  
                }    
          
        }
        if(cont == 9)
            while(analogRead(v2) > 100);
        return ler;         // retorna valor entre 0 ...1023

}

float valores(int op) // 4 - seleciona caracters validos, Backspace, somente 2 casas decimais, numero com no máximo 6 digitos. 
{

    char key;
    char keyax;
    char key1={};
    String num = "", num1 = "";
    int pt=0, zz=0;
    int contador=0;
    int seis=0;
    
  while (key != 13){  
  
              
                
              key = ler_teclado(pt,op);
              delay(200);
              
              
              if (key =='.')
                key1 = key;
  
                               
               if ((key== '/') || (key == '*') || (key == '+') || (key=='-')||(key== 'w')){
               
               }

               else if (key == 'B'){
                
                if(seis>0){
                   
                   seis--;
                   
                   num.remove(seis,1);
                   if(keyax=='.'){
                        lc.setChar(op,6-seis,num[seis-1],false);
                       //Serial.println();
                       //Serial.println(num);
                        key1={};
                        contador --;
                      
                      
                   }
                   else {
                       lc.setChar(op,5-seis,' ',false);
                      //Serial.println();
                      //Serial.println(num);
                       lc.setChar(op,4-seis,' ',false); 
                       if(contador>0)
                        contador --;
                       pt--;
                   }
                }   
               }
              
              else if((contador == 1) && (key == '.')){
                }
                   
              else if((contador >0) && (key =='.')){
                }
  
              else if((key!=0) && (key!= '+') && (key != '-') && (key != '/') && (key!= 13)){
                
                  if(contador <= 2){
                     num.concat(key);
                     //Serial.print(num);

                            
                     if(key =='.')
                        lc.setChar(op,6-pt,num[seis-1],true);
                     else{
                        lc.setChar(op,5-pt,key,false);
                        pt++;                      
                      }
                   if(key1 == '.')
                        contador++;
                        seis++;      
                  }
              }
              keyax = key;
              if (seis ==6)//numero com no máximo 6 digitos
                break;
              if(contador==3) //2 casa decimais
                break;  
    }
    

    //Serial.println();
    //Serial.println(num);
    lc.setRow(op,5-seis,B0000000);
    if (contador> 1)
      lc.setRow(op,6-seis,B0000000);
    
    return num.toFloat();
} 

int *InsertionSort(int original[], int length) // Algoritmo já conhecido para organizar um vetor em ordem crescente
{
  int i, j, atual;

  for (i = 1; i < length; i++)
  {
    atual = original[i];
    j = i - 1;

    while ((j >= 0) && (atual < original[j]))
    {
      original[j + 1] = original[j];
            j = j - 1;
    }    
    original[j + 1] = atual;
  }
  return (int*)original;
}


void text1()//Calculo de IMC
{
    
    lc.setRow(0,7,B1001110);
    lc.setRow(0,6,B1110111);
    lc.setRow(0,5,B0001110);
    lc.setRow(0,4,B1001110);
    lc.setRow(0,3,B0111110);
    lc.setRow(0,2,B0001110);
    lc.setRow(0,1,B1111110);
    lc.setRow(1,7,B0111101);
    lc.setRow(1,6,B1001111);
    lc.setRow(1,4,B0110000);
    lc.setRow(1,3,B1100110);
    lc.setRow(1,2,B1110010);
    lc.setRow(1,1,B1001110);
    delay(1500); 
}

void text2()// IFPB  -- Joao Pessoa
{
        //ifpb
        lc.setRow(0,5,B0110000);
        lc.setRow(0,4,B1000111);
        lc.setRow(0,3,B1100111);
        lc.setRow(0,2,B0011111);

        //Joao Pessoa
        for(int xi = 0;xi <4;xi++){
            for (int zi=0; zi<8;zi++) 
                lc.setRow(1,zi,B0000000);
            lc.setRow(1,7+xi,B1111100);
            lc.setRow(1,6+xi,B1111110);
            lc.setRow(1,5+xi,B1110111);
            lc.setRow(1,4+xi,B1111110);
    
            lc.setRow(1,2+xi,B1100111);
            lc.setRow(1,1+xi,B1001111);
            lc.setRow(1,0+xi,B1011011);
            lc.setRow(1,-1+xi,B1011011);
            lc.setRow(1,-2+xi,B1111110);
            lc.setRow(1,-3+xi,B1110111);

            if(xi == 0)
                delay(500);

            delay(300);
        }
        delay(500);
}

void text3()// Felipe Guedes    Giovanna Dalia
{
   for ( int xi=0;xi<9;xi++){
        //Felipe
        limpaa();
        lc.setRow(0,7-xi,B1000111);
        lc.setRow(0,7-xi-1,B1001111);
        lc.setRow(0,7-xi-2,B0001110);
        lc.setRow(0,7-xi-3,B0110000);
        lc.setRow(0,7-xi-4,B1100111);
        lc.setRow(0,7-xi-5,B1001111);
        //Guedes
        lc.setRow(1,7-xi+0,B1011111);
        lc.setRow(1,7-xi-1,B0111110);
        lc.setRow(1,7-xi-2,B1001111);
        lc.setRow(1,7-xi-3,B0111101);
        lc.setRow(1,7-xi-4,B1001111);
        lc.setRow(1,7-xi-5,B1011011);
        if (xi ==0)
            delay(500);      

        delay(200);
   } 
    for ( int xi=0;xi<9;xi++){ 
        limpaa();
           //Giovanna
        lc.setRow(0,7-xi-0,B1011111);
        lc.setRow(0,7-xi-1,B0110000);
        lc.setRow(0,7-xi-2,B1111110);
        lc.setRow(0,7-xi-3,B0111110);
        lc.setRow(0,7-xi-4,B1110111);
        lc.setRow(0,7-xi-5,B0010101);
        lc.setRow(0,7-xi-6,B0010101);
        lc.setRow(0,7-xi-7,B1110111);
          //Dalia
        lc.setRow(1,7-xi-0,B0111101);
        lc.setRow(1,7-xi-1,B1110111);
        lc.setRow(1,7-xi-2,B0001110);
        lc.setRow(1,7-xi-3,B0110000);
        lc.setRow(1,7-xi-4,B1110111);
        if (xi ==0)
            delay(500);  
       delay(200);   
    }
    delay(500);
}
void text4()//Digite    Peso e Altura
{       
        limpaa();
        //digite
        lc.setRow(0,7,B0111101);
        lc.setRow(0,6,B0110000);
        lc.setRow(0,5,B1011111);
        lc.setRow(0,4,B0110000);
        lc.setRow(0,3,B0001111);
        lc.setRow(0,2,B1001111);
        delay(1000);
        limpaa();

        //peso
        lc.setRow(0,7,B1100111);
        lc.setRow(0,6,B1001111);
        lc.setRow(0,5,B1011011);
        lc.setRow(0,4,B1111110);
        lc.setRow(0,1,B1001111);
        

        // altura
        lc.setRow(1,7,B1110111);
        lc.setRow(1,6,B0001110);
        lc.setRow(1,5,B0001111);
        lc.setRow(1,4,B0111110);
        lc.setRow(1,3,B0000101);
        lc.setRow(1,2,B1110111);
        delay(1000);
        for (int ri=0; ri<6; ri++){
            lc.setRow(0,ri,B0000000);
            lc.setRow(1,ri,B0000000);
            delay(200);
        }
        
          
}

void text5()// P=    A=
{
    lc.setRow(0,7,B1100111);
    lc.setRow(0,6,B1000001);
    lc.setRow(1,7,B1110111);
    lc.setRow(1,6,B1000001);
    
}
void timc()// IMC =
{

    lc.setRow(0,6,B0110000);
    lc.setRow(0,5,B1100110);
    lc.setRow(0,4,B1110010);
    lc.setRow(0,3,B1001110);
    lc.setRow(0,1,B1001000);
  
}

void magreza_II()
{
    limpaa();
    lc.setRow(0,7,B1100110);
    lc.setRow(0,6,B1110010);
    lc.setRow(0,5,B1110111);
    lc.setRow(0,4,B1011111);
    lc.setRow(0,3,B0000101);
    lc.setRow(0,2,B1001111);
    lc.setRow(0,1,B1101101);
    lc.setRow(0,0,B1110111);

    lc.setChar(1,7,'2',false);
    
}
void magreza_I()
{

    limpaa();
    lc.setRow(0,7,B1100110);
    lc.setRow(0,6,B1110010);
    lc.setRow(0,5,B1110111);
    lc.setRow(0,4,B1011111);
    lc.setRow(0,3,B0000101);
    lc.setRow(0,2,B1001111);
    lc.setRow(0,1,B1101101);
    lc.setRow(0,0,B1110111);

    lc.setChar(1,7,'1',false); 
   
}

void normal()
{
    limpaa();
    lc.setRow(0,7,B0010101);
    lc.setRow(0,6,B1111110);
    lc.setRow(0,5,B0000101);
    lc.setRow(0,4,B1100110);
    lc.setRow(0,3,B1110010);
    lc.setRow(0,2,B1110111);
    lc.setRow(0,1,B0001110);
}

void acima_do_peso()
{
    limpaa();
    lc.setRow(0,7,B1110111);
    lc.setRow(0,6,B1001110);
    lc.setRow(0,5,B0110000);
    lc.setRow(0,4,B1100110);
    lc.setRow(0,3,B1110010);
    lc.setRow(0,2,B1110111);
    
    lc.setRow(1,7,B0111101);
    lc.setRow(1,6,B1111110);
    lc.setRow(1,5,B0000000);
    lc.setRow(1,4,B1100111);
    lc.setRow(1,3,B1001111);
    lc.setRow(1,2,B1011011);
    lc.setRow(1,1,B1111110);
      
}

void obesidade_I()
{
    limpaa();
    lc.setRow(0,7,B1111110);
    lc.setRow(0,6,B0011111);
    lc.setRow(0,5,B1001111);
    lc.setRow(0,4,B1011011);
    lc.setRow(0,3,B0110000);
    lc.setRow(0,2,B0111101);
    lc.setRow(0,1,B1110111);
    lc.setRow(0,0,B0111101);

    lc.setRow(1,7,B1001111);
    
    lc.setChar(1,5,'1',false); 
}

void obesidade_II()
{
    limpaa();
    lc.setRow(0,7,B1111110);
    lc.setRow(0,6,B0011111);
    lc.setRow(0,5,B1001111);
    lc.setRow(0,4,B1011011);
    lc.setRow(0,3,B0110000);
    lc.setRow(0,2,B0111101);
    lc.setRow(0,1,B1110111);
    lc.setRow(0,0,B0111101);

    lc.setRow(1,7,B1001111);
    
    lc.setChar(1,5,'2',false); 
}

void obesidade_III()
{
    limpaa();
    lc.setRow(0,7,B1111110);
    lc.setRow(0,6,B0011111);
    lc.setRow(0,5,B1001111);
    lc.setRow(0,4,B1011011);
    lc.setRow(0,3,B0110000);
    lc.setRow(0,2,B0111101);
    lc.setRow(0,1,B1110111);
    lc.setRow(0,0,B0111101);

    lc.setRow(1,7,B1001111);
    
    lc.setChar(1,5,'3',false); 
}


void limpaa() //  limpa todos os displays
{
    for (int xi=0; xi <2; xi++){
        for (int zi=0; zi<8;zi++) 
            lc.setRow(xi,zi,B0000000);
    }
}
