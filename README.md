# Esteira

__Equipe:__ Webert Cristhian Dutra Soares e Gabriel Tosta Nunes

Os sensores de esteira são dispositivos fundamentais para sistemas industriais e de
automação, sendo empregados para monitorar e controlar processos ao longo de linhas de
produção. Neste projeto, estamos focando na implementação e aprimoramento de quatro
tipos de sensores: o sensor de velocidade, o sensor de cor, o sensor de temperatura e o
sensor de ultrassom. As melhorias visam otimizar o desempenho da esteira, garantir maior
precisão na medição, e assegurar uma produção mais eficiente e segura.

A melhoria na implementação dos sensores de esteira é essencial para acompanhar as
demandas de eficiência e qualidade da indústria moderna. Sensores mais precisos e
otimizados permitem um controle mais fino da operação, reduzindo o tempo de parada,
melhorando a qualidade dos produtos e minimizando perdas. A integração adequada
desses sensores também garante maior segurança, pois possibilita a identificação rápida de
problemas, além de oferecer dados em tempo real para a tomada de decisão gerencial.

O objetivo geral do projeto é entregar uma informação útil para o processamento, além de 
um perfeito funcionamento da esteira em conjunto com os sensores. Então basicamente 
verificaremos o funcionamento de cada sensor individualmente, acarretando em tarefas da 
seguinte ordem:

Entender o funcionamento do controle do motor da esteira;
Ajustar e calibrar o sensor de velocidade afim de colocar o dispositivo em funcionamento;
Revisar a informação de saída dos sensores de cor, ultrasson e temperatura;

Sensor de Velocidade: Monitoramento e controle da velocidade da esteira para garantir
que o ritmo da produção seja eficiente. As melhorias buscam aumentar a precisão na
resposta e evitar sobrecarga no sistema.

Sensor de Cor: Utilizado para identificar e classificar produtos com base em suas cores.
As melhorias visam aumentar a sensibilidade do sensor, garantindo que nenhuma anomalia
passe despercebida.

Sensor de Temperatura: Controla a temperatura de máquinas e produtos, evitando
superaquecimento e garantindo a qualidade. Com as atualizações, espera-se maior
precisão e respostas mais rápidas às variações.

Sensor de Ultrassom: Mede distâncias e detecta obstáculos ao longo da linha de
produção. As melhorias neste sensor permitirão uma detecção mais eficiente de objetos,
otimizando o fluxo produtivo e evitando paradas inesperadas.

![Esteira](https://raw.githubusercontent.com/WebertCris/Esteira/refs/heads/main/figuras/Esteira.jpg)

## Design

Neste projeto de automação, o sensor de velocidade e o sensor de cor são fundamentais para o 
monitoramento e controle do fluxo produtivo na esteira. Ambos os sensores foram escolhidos 
para garantir precisão e estabilidade na operação, promovendo eficiência e controle de 
qualidade em tempo real. Cada sensor conta com um módulo ESP32 dedicado, permitindo 
processamento independente, fácil ajuste e comunicação eficiente com o sistema central. 
Nosso Objetivo Principal aqui, é a implementação de soluções para os problemas enfrentados 
anteriormente pelos projetores dos devidos sensores e o funcionamento do motor. 
Como a princípio estaremos trabalhando apenas com programação, o hardware que controla 
todo esse monitoramento da esteira se mantém.


Protocolo MQTT: O protocolo MQTT (Message Queuing Telemetry Transport) é uma tecnologia de comunicação 
leve, projetada para transmitir dados entre dispositivos em redes de baixa largura de banda 
e conexões instáveis, sendo amplamente utilizado em aplicações de Internet das Coisas (IoT). 
Criado pela IBM, o MQTT foi desenvolvido para oferecer uma solução simples e eficiente para 
cenários de comunicação máquina-a-máquina (M2M). Ele utiliza um modelo de publicação/assinatura, 
no qual os dispositivos não se comunicam diretamente entre si. Em vez disso, cada dispositivo 
pode publicar mensagens em "tópicos" ou assinar tópicos específicos para receber mensagens. 
Um "broker" MQTT (servidor intermediário) gerencia essas mensagens e as distribui aos dispositivos 
corretos, de acordo com os tópicos que estão sendo assinados.

Controle do Motor: L298N: Um circuito integrado utilizado como controlador de motores DC bidirecionais, 
e que suporta uma corrente alta, o CI consiste em duas pontes H, permitindo o controle independente de dois 
motores. Ele possui entrada de controle por sinais digitais (geralmente PWM) que determinam a velocidade e 
a direção de rotação dos motores.
ESP-32: É o microcontrolador do projeto, sendo responsável pela leitura dos valores dos sensores, por controlar 
os atuadores e através do protocolo MQTT receber o publicar os tópicos que sejam relacionados.
Broker: realiza a comunicação entre os microcontroladores através do protocolo MQTT.
Motor DC, MR 710-ISVBUN-12-24V: motor que será usado na esteira, é fabricado pela Motron, sendo um Motoredutor de 
corrente contínua a Tensão de armadura de 24V e uma velocidade de 12RPM.
Sensor de velocidade encoder LM393. : Um sensor de velocidade ira "medir" a velocidade da rotação do motor e assim 
será possível calcular a velocidade da esteira.

Especificações do Motor
Tensão de armadura de 24v
Ambos os sentidos de rotação
Quanto maior a carga maior a corrente, mas o torque diminui
"Os moto-redutores suportam os torques e as correntes máximas por 15 segundos, sob riscos de danos permanentes"
Classe de isolação F 155° C (temperatura máxima de operação)
Carga radial máxima de 3kg e axial de 1,5kg
todos possuem as mesmas dimensões e peso de 0.64kg

Problemas e Melhorias no Sensor de Velocidade

O sensor de velocidade, baseado no LM393, monitora a rotação do motor e fornece informações em RPM. 
Alguns problemas identificados na implementação do sensor de velocidade incluem: 
Bloqueio da Função "Acelera": A função atual, que usa "delay" para controlar a aceleração, bloqueia 
o código enquanto espera a velocidade atingir o ponto desejado. Durante esse tempo, outras leituras 
e comandos, como o monitoramento de RPM e as mensagens MQTT, não podem ser processados.
Controle Seguro de Inversão de Sentido: Uma mudança brusca no sentido de rotação do motor pode causar 
desgaste ou danos. Implementaremos um controle que detecta mudanças na variável de sentido e primeiro 
desacelera o motor até zero antes de inverter o sentido.

Inconsistência na Medição do RPM: Em alguns testes, o sistema apresentou medições incorretas de RPM, 
mesmo sem conexão com o sensor encoder, indicando uma possível interferência ou erro de código. 
Publicação da Velocidade Real (km/h): Em vez de depender apenas do valor de RPM, o sistema agora 
publicará a velocidade real da esteira em km/h, utilizando o valor de RPM ajustado à circunferência 
do eixo do motor.
Funcionamento do Sensor de Cor.

O Sensor de cor TCS34725, representado na figura 02, utiliza uma matriz de fotodiodos sensíveis à 
luz vermelha, verde, azul e infravermelha para capturar a cor de uma superfície ou objeto. Esses 
fotodiodos convertem a luz incidente em corrente elétrica, que é então convertida em sinais digitais 
para determinar a intensidade de cada cor. Uma das vantagens do TCS34725 é sua capacidade de compensar 
a luz ambiente, permitindo medições precisas mesmo em diferentes condições de iluminação. Além disso, 
ele possui um filtro de luz infravermelha embutido, que ajuda a melhorar a precisão das medições de cor.
Problemas e Melhorias no Sensor de Cor.

Precisão na Leitura de Cores Fora do Espectro RGB: Cores que não pertencem estritamente aos parâmetros 
configurados no código (rosa, laranja, vermelho, verde, azul), apresentam desafios de detecção e podem 
resultar em leituras incorretas devido à proximidade com os espectros vermelho e verde. Para solucionar 
isso, faremos ajustes nos parâmetros de calibração, buscando aumentar a sensibilidade do sensor TCS34725 
para identificar nuances e tons intermediários com maior precisão, buscando utilizar a escala RGBC.

Objetivos: Para os próximos testes no projeto da esteira com quatro sensores, será essencial realizar uma verificação 
detalhada do funcionamento individual de cada sensor. Esses testes devem assegurar que cada sensor opere 
de forma independente e confiável, captando dados precisos e consistentes. Além disso, será necessário 
avaliar a qualidade das informações coletadas por cada um, a fim de garantir que os dados enviados ao banco 
estejam dentro dos padrões exigidos para processamento e análise. Esse processo permitirá identificar possíveis 
inconsistências e otimizar o sistema, assegurando que a esteira esteja pronta para uma operação eficaz e de 
alto desempenho.

Referências


