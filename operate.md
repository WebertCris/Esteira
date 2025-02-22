## Conclusão

Durante o desenvolvimento deste projeto, conseguimos estabelecer com sucesso a comunicação dos sensores da esteira com o Home Assistant 
por meio do protocolo MQTT. Enfrentamos desafios devido às restrições do Home Assistant, que não permite publicação e inscrição no mesmo 
tópico disponibilizado. Como solução, optamos por fazer com que o ESP apenas se inscrevesse no tópico do motor da esteira e, com o uso de 
um knob virtual, cuja resposta é enviada em um JSON com as chaves module e value, conseguimos controlar a potência da esteira de maneira 
eficiente.

Além disso, aprimoramos o código do sensor de cor para que a resposta enviada ao Home Assistant fosse um código hexadecimal correspondente 
à cor detectada. Dessa forma, estruturamos a mensagem em JSON com as chaves sensor, função e valor, garantindo que o valor enviado fosse o 
código hexadecimal correto. Com essa abordagem, conseguimos uma representação mais precisa da cor no Home Assistant, aprimorando a 
usabilidade e a confiabilidade do sistema.

Esse projeto demonstrou a viabilidade da integração de sensores com o Home Assistant e permitiu um melhor entendimento sobre 
comunicação via MQTT, tratamento de dados e automação. O aprendizado adquirido poderá ser aplicado em futuras implementações e 
melhorias no sistema.

Para o próximo semestre, sugerimos como continuidade o desenvolvimento de uma nova biblioteca para o controle do motor da esteira, 
integrando também um sensor de velocidade. Essa melhoria permitirá um controle mais preciso e flexível, além de otimizar o desempenho 
geral do sistema e ampliar as funcionalidades do projeto.
