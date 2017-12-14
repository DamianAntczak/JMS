Java Message Service   
===

Instrukcja przedstawia przykładowe wykorzystanie Java Message Service


# Aplikacja Hello World

## Konfiguracja projektu

1. Utwórz nowy projekt w `IDE Intellij IDEA` o nazwie `JMS` ze wsparciem `Maven`.

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
    - Po dodaniu zależności zaimportuj zmiany.

## Implementacj nadawcy

1. Utwórz pakiet `point2point` i dodaj do niego klasę `Sender.java`. Jej zadaniem będzie:
    - Połączenie się do brokera `ActiveMQ`,
    - Utworzenie kolejki o nazwie `BSR`,
    - Wysłanie za pomocą obiektu klasy `MessageProducer` tekstowej wiadomości do kolejki.

2. Dodaj prywatne pole klasy typu `Connection`:
    ``````````
    private Connection connection;
    ``````````

3. Utwórz publiczny konstruktor klasy:
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

4. Dodaj publiczną metodę close:
    ``````````````
    public void close() throws JMSException {
        connection.close();
    }
    ``````````````

## Implementacja odbiorcy
1. Dodaj do projektu nową klasę `Consumer.java` implementującą interfejs `MessageListener`. Jej zadaniem będzie:
    - Połączenie się do brokera `ActiveMQ`,
    - Utworzenie kolejki o nazwie `BSR`,
    - Implementacja metody `onMessage`, która będzie odczytywać wiadomość z kolejki:
    
2. Dodaj prywatne pole `Connection connection`.
    
3. Zaimplementuj metodę `public void onMessage(Message message)` z interfejsu `JMSException`:
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
   
4. Zaimplementuj metodę `startListener()`, która będzie inicjalizowała połączenie. Metoda ta będzie rzucała wyjątek `JMSException`:
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
    
5. Dodaj publiczną metodę close:
    ``````````````
    public void close() throws JMSException {
        connection.close();
    }
    ``````````````
    
## Uruchomienie aplikacji

1. Dodaj klasę `Main` z statyczną metodą `public static void main(String[] args)`.

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
   
## Rozszerzenie aplikacji

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

 
3. Z wykorzystaniem wyciętego kodu stwórz metodę `public void sendMessage (String message) throws JMSException`, która będzie wysyła wiadomość podaną w argumecie `message`. Pamiętaj o zaincjalizowaniu obiektów klasy `Session` i `MessageProducer` (Zamień odpowiednie zmienne lokalne na pola)

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
    
# Aplikacja typu Opublikuj/Subskrybuj

## Implementacja klasy Publisher

1. Dodaj do projektu nowy pakiet `subscription`

2. Dodaj do nowo utworzonego pakietu klasę `Publisher.java`

3. Dodaj do klasy poniższe pola:
    ````
    private Connection connection;
        private Session session;
        private MessageProducer messageProducer;
    ````

3. Utwórz wieloparametrowy konstruktor:
    ``````
    public Publisher(String clientId, String topicName) throws JMSException {
    
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("vm://localhost");

        connection = connectionFactory.createConnection();
        connection.setClientID(clientId);

        session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(topicName);
        messageProducer = session.createProducer(topic);
    }
    ``````

4. Dodaj publiczną metodę `close()` taką samą jak we wcześniejszych klasach

5. Dodaj metodę `sendName`, za pomocą której do `topic` zostanie wysłana wiadomość
    ```
    public void sendName(String firstName, String lastName) throws JMSException {
        String text = firstName + " " + lastName;
    
        TextMessage textMessage = session.createTextMessage(text);
        messageProducer.send(textMessage);
    
        System.out.println(this.clientId + ": wysłanie wiadomości o treści=" + text);
    }
    ```

## Implementacja klasy Subscriber

1. Dodaj do pakietu `subscription` klasę `Subscriber`

2. Dodaj do klasy poniższe pola oraz konstruktor: 
    ```````
    private String clientId;
    private Connection connection;
    private Session session;
    private MessageConsumer messageConsumer;
    
    public Subscriber(String clientId, String topicName) throws JMSException {
        this.clientId = clientId;
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("vm://localhost");
    
        connection = connectionFactory.createConnection();
        connection.setClientID(clientId);
    
        session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    
        Topic topic = session.createTopic(topicName);
    
        messageConsumer = session.createConsumer(topic);
        connection.start();
    }
    ```````

3. Dodaj metodę `getGreeting`, która będzie pobierać wiadomość z tematu i zwracać pozdrowienie
    ``````
    public String getGreeting(int timeout) throws JMSException {
    
        String greeting = "Brak pozdrowień";
        Message message = messageConsumer.receive(timeout);
    
        if (message != null) {
            TextMessage textMessage = (TextMessage) message;
    
            String text = textMessage.getText();
            System.out.println(clientId + ": odbiór wiadomości o treści=" + text);
    
            greeting = "Hello " + text + "!";
        } else {
            greeting = clientId + ": żadna wiadomość nie została odebrana";
        }
        return greeting;
    }
    ``````
    
4. Dodaj klasę `SubscriberExample`, która będzie demonstrowała działanie modelu wydawca/subskrybent. Klasa ta będzie posiadać jednego wydawce i dwóch odbiorców.

5. Dodaj do klasy metodę `main`.
    ``````
        public static void main(String[] args) throws JMSException {
    
            BasicConfigurator.configure();
            Logger.getRootLogger().setLevel(Level.OFF); //commenting this line enables log
    
            Publisher publisher = new Publisher("publisher1", "bsr.topic");
    
            Subscriber subscriber1 = new Subscriber("subscriber1", "bsr.topic");
            Subscriber subscriber2 = new Subscriber("subscriber2", "bsr.topic");
    
    
            publisher.sendName("Harry", "Potter");
            publisher.sendName("Jack", "Sparrow");
    
            String greeting = subscriber1.getGreeting(1000);
            System.out.println(greeting);
            String greeting1 = subscriber2.getGreeting(1000);
            System.out.println(greeting1);
    
            System.out.println(subscriber2.getGreeting(1000));
            System.out.println(subscriber2.getGreeting(1000));
    
            subscriber1.close();
            subscriber2.close();
            publisher.close();
        }
    ``````
    
7. Uruchom przykład z klasy `SubscriberExample`.
    
6. Pytanie:
    - Co stanie się po dodaniu kolejnego obiektu klasy `Publisher` z parametrem `topicName="bsr.topic"` i wysłaniu `sendName("Pippi","Langstrump")`?

    - Sprawdź to zmieniając implementacja `main` w klasie `SubscriberExample`