# WEBSOCKETS
protocol for bi-directional client-server communication

## Annotated Endpoints

```
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
