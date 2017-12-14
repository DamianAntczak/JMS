Java Message Service   
===

Instrukcja przedstawia przykładowe wykorzystanie Java Message Service


#Aplikacja Hello World

##Konfiguracja projektu

1. Utwórz nowy `Empty Project` w `IDE Intellij IDEA` o nazwie `JMS`.

2. Dodaj do projektu modół o nazwie `'

2. Dodaj do pliku `pom.xml` poniższe zależności:
    ```````````
    <dependencies>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-core</artifactId>
            <version>5.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.5</version>
        </dependency>
    </dependencies>
    ```````````
Po dodaniu zależności zaimportuj zmiany

##Implementacj nadawcy

1. Dodaj do projektu nową klasę `Sender.java`. Jej zadaniem będzie:
    - Połączenie się do brokera `ActiveMQ`
    - Utworzenie kolejki o nazwie `BSR`
    - Wysłanie za pomocą obiektu klasy `MessageProducer` tekstowej wiadomości do kolejki

2. Dodaj prywatne pole klasy typu `Connection`
    ``````````
    private Connection connection;
    ``````````

3. Utwórz publiczny konstruktor klasy
    `````````````
    public  Sender() throws JMSException {
        ConnectionFactory connectionFactory =  new ActiveMQConnectionFactory("vm://localhost");
        connection = connectionFactory.createConnection();
        connection.start();
    
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue("BSR");
        MessageProducer producer = session.createProducer(queue);
    
        TextMessage textMessage = session.createTextMessage("Hello JMS!!!");
        System.out.printf("Wysyłam wiadomość: %s, Wątek: %s%n", textMessage.getText(), Thread.currentThread().getName());
        producer.send(textMessage);
    }
    `````````````

4. Dodaj publiczną metodę close
    ``````````````
    public void close() throws JMSException {
        connection.close();
    }
    ``````````````

##Implementacja odbiorcy
1. Dodaj do projektu nową klasę `Consumer.java` implementującą interfejs `MessageListener`. Jej zadaniem będzie:
    - Połączenie się do brokera `ActiveMQ`
    - Utworzenie kolejki o nazwie `BSR`
    - Implementacja metody `onMessage`, która będzie odczytywać wiadomość z kolejki
    
2. Dodaj prywatne pole `Connection connection`
    
3. Zaimplementuj metodę `public void onMessage(Message message)` z interfejsu `JMSException`
    ````
    public void onMessage(Message message) {
        if(message instanceof TextMessage){
            TextMessage tm = (TextMessage) message;
            try{
                System.out.printf("Wiadomość odebrana: %s, Wątek: %s%n", tm.getText(),Thread.currentThread().getName());
            }
            catch (JMSException e){
                throw new RuntimeException(e);
            }
        }
    }
    ````
   
4. Zaimplementuj metodę `startListener()`, która będzie inicjalizowała połączenie. Metoda ta będzie rzucała wyjątek `JMSException`
    ````````
    public void startListener() throws JMSException{
        ConnectionFactory factory = new ActiveMQConnectionFactory("vm://localhost");
        connection = factory.createConnection();
        connection.start();
    
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    
        Queue queue = session.createQueue("BSR");
        MessageConsumer consumer = session.createConsumer(queue);
        consumer.setMessageListener(this);
    }
    ````````
    
5. Dodaj publiczną metodę close
    ``````````````
    public void close() throws JMSException {
        connection.close();
    }
    ``````````````
    
##Uruchomienie aplikacji

1. Dodaj klasę `Main` z statyczną metodą `public static void main(String[] args)`

2. Zaimplementują w metodzie `main` obsługę klas `Sender` i `Reciver`:
    ````
    BasicConfigurator.configure();
    Logger.getRootLogger().setLevel(Level.OFF); //commenting this line enables log
    
    try {
        Sender sender = new Sender();
    
        Thread.sleep(2000);
    
        Receiver receiver = new Receiver();
        receiver.startListener();
    
        sender.close();
        receiver.close();
    }
    catch (JMSException |InterruptedException e){
        e.printStackTrace();
    }
    ````
    
3. Uruchom aplikacje
   - po wykonaniu powyższy kroków powinniśmy otrzymać komunikaty: `Wysyłam wiadomość: Hello JMS!!!, Wątek: main`, a następnie po 2 sekundach `Wiadomość odebrana: Hello JMS!!!, Wątek: ActiveMQ Session Task-1`
   
##Rozszerzenie aplikacji

1. Dodaj do klasy pola:
    ``````````
    private Session session;
    private MessageProducer producer;
    ``````````

2. W klasie `Sender` wytnij, poniższe linijki:
    ``````````
    TextMessage textMessage = session.createTextMessage("Hello JMS!!!");
    System.out.printf("Wysyłam wiadomość: %s, Wątek: %s%n", textMessage.getText(), Thread.currentThread().getName());
    producer.send(textMessage);
    ``````````

 
3. Z wykorzystaniem wyciętego kodu stwórz metodę `public void sendMessage (String message) throws JMSException`, która będzie wysyła wiadomość podaną w argumecie `message`. Pamiętaj o zaincjalizowaniu obiektów klasy `Session` i `MessageProducer` (Zamień odpowienie zmienne lokalne na pola)

4. W klasie `Main` zamień sekcji `try` na poniższy kod:
    ``````
    Sender sender = new Sender();
    Receiver receiver = new Receiver();
    receiver.startListener();

    for (int i = 1; i <= 5; i++) {
        sender.sendMessage("Hello world! " + i);
        Thread.sleep(1000);
    }

    sender.close();
    receiver.close();
    ``````
    - Jakie wnioski zaobserwowałeś?
    
#Aplikacja typu Opublikuj/Subskrybuj

##Dodanie nowego pakietu
