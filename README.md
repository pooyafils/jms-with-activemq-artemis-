# jms-with-activemq-artemis
## Overview
ActiveMQ Artemis is an open-source "next generation" broker from Apache with the performance and feature-set to implement high-performance messaging systems. Artemis is the code-name used for the HornetQ code that was donated to the Apache Foundation.
## Quick set up
1.install activemq-artemis in docker by ``docker pull vromero/activemq-artemis``<br/>
2.you need to open cmd and type ``docker run -it --rm  -p 8161:8161 -p 61616:61616 vromero/activemq-artemis``
3.enter url ``http://localhost:8161/console/auth/login``. username is -> artemis and password is -> simetraechcapa
## Development note
for this repository we used springboot to develop applications to send and receive messages
1. add maven dependency
```
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-artemis</artifactId>
		</dependency>
```
2. adding configuration for a queue and sending message
```
@EnableScheduling
@EnableAsync
@Configuration
public class JmsConfig {
    public static final String MY_QUEUE="hello word";
    public static final String MY_REPLAY_QUEUE="replay to me";
    @Bean
    public MessageConverter messageConverter(){
        MappingJackson2MessageConverter converter=
                new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        return converter;
    }
}
```
3. we need to have ``JmsTemplate`` to send the message 

```
@Component
public class MyMessageSender {
    private final JmsTemplate jmsTemplate;
    private final ObjectMapper objectMapper;

    public MyMessageSender(JmsTemplate jmsTemplate, ObjectMapper objectMapper) {
        this.jmsTemplate = jmsTemplate;
        this.objectMapper = objectMapper;
    }
     @Scheduled(fixedRate = 5000)
    public void sendMessage(){
         System.out.println("i am sending message");

        MyMessage myMessage=MyMessage.builder()
                .id(UUID.randomUUID())
                .myMessage("Hello")
                .build();
        System.out.println("    Message send");
        jmsTemplate.convertAndSend(JmsConfig.MY_QUEUE,myMessage);

    }
```
we have to inject the ``JmsTemplate`` for send the message to the Listener 
4. we have to write the configuration for the message queue
```
@EnableScheduling
@EnableAsync
@Configuration
public class JmsConfig {
    public static final String MY_QUEUE="hello word";
    public static final String MY_REPLAY_QUEUE="replay to me";
    @Bean
    public MessageConverter messageConverter(){
        MappingJackson2MessageConverter converter=
                new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        return converter;
    }
}

```
5. in the last step we develop a jms listener to receive the message and for this reason we use ``@JmsListener(destination = JmsConfig.MY_QUEUE)`` which specify  which queue is for  our listener method
```
   @JmsListener(destination = JmsConfig.MY_QUEUE)
       public void listener(@Payload MyMessage myMessage, @Headers MessageHeaders messageHeaders
            , Message meaage){
       System.out.println("i got the message");
        System.out.println(myMessage);

```
