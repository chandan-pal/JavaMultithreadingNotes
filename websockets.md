# WEBSOCKETS
protocol for bi-directional client-server communication

## Annotated Endpoints

```java
@ServerEndpoint("/chat") // javax.websocket.server
public class ChatEndPoint {
  private static final ConcurrentLinkedQueue<Session> peers = new ConcurrentLinkedQueue<>();
  
  @OnOpen
  public void open(Session session) {
    System.out.println("New session opened!");
    peers.add(session);
  }
  
  @OnClose
  public void close(Session session, CloseReason closeReason) {
    System.out.println("Session closed with reason - " + closeReason.getReasonPhrase());
    peers.remove(session);
  }
  
  @OnMessage
  public void relayMessage(String message, Session session) throws Exception {
    for (Session peer: peers) {
      if (!peer.equals(session)) {
        peer.getBasicRemote().sendText(message);
      }
    }
  }
}
```

## Programmatic Endpoints
```java
public class ServerConfig implements ServerApplicationConfig {
  @Override
  public Set<ServerEndpointConfig> getEndpointConfigs(Set<Class<? extends Endpoint>> set {
    return new HashSet<ServerEndpointConfig>() {
      add(ServerEndpointConfig.Builder.create(MyProgrammaticEndpoint.class, "/chat").build());
    };
  }
}


public class MyProgrammaticEndpoint extends Endpoint {
  @Override
  public void onOpen(Session session, EndpointConfig endpointConfig) {
    session.addMessageHandler((MessageHandler.Whole<String>) s -> {
      System.out.println("Server : " + s);
      try {
        session.getBasicRemote().sendText("response message");
      } catch (IOException ex) {
        System.out.println("ERROR:");
      }
    });
  }
}
```

## Encoder and Decoder
Encoders and Decoders are used in websockets to transform the POJOs to response and vice versa. Websokets API does not do automatic marshelling and demarshelling.

```java
public class MyMessageEncoder implements Encoder.Text<MyMessage> {
  @Override
  public String encode(MyMessage myMessage) {
    return JsonBuilder.create().toJson(myMessage);
  }
  
  @Override
  public void init(EndpointConfig endpointConfig) {
    
  }
  
  @Override
  public void destroy() {
    
  }
}

public class MyMessageDecoder implements Decoder.Text<MyMessage> {
  @Override
  public String decode(String strMessage) {
    return JsonBuilder.create().fromJson(strMessage, MyMessage.class);
  }
  
  @Override
  public boolean willDecode(String s) {
    return true;
  }
  
  @Override
  public void init(EndpointConfig endpointConfig) {
    
  }
  
  @Override
  public void destroy() {
    
  }
}


// register encoder and decoder at endpoints
@ServerEndpoint("/chat", encoders=MyMessageEncoder.class, decoders=MyMessageDecoder.class) // register encoder
public class ChatEndPoint {
  private static final ConcurrentLinkedQueue<Session> peers = new ConcurrentLinkedQueue<>();
  
  @OnOpen
  public void open(Session session) {
    System.out.println("New session opened!");
    peers.add(session);
  }
  
  @OnClose
  public void close(Session session, CloseReason closeReason) {
    System.out.println("Session closed with reason - " + closeReason.getReasonPhrase());
    peers.remove(session);
  }
  
  @OnMessage
  public void relayMessage(MyMessage message, Session session) throws Exception {
    for (Session peer: peers) {
      if (!peer.equals(session)) {
        peer.getBasicRemote().sendObject(message); //sending object here...encoder will encode it to text.
      }
    }
  }
}
```
