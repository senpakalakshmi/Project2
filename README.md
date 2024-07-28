import java.util.HashMap;
import java.util.Map;

import java.util.ArrayList;
import java.util.List;
import java.time.LocalDateTime;

class User {
    private String username;
    private List<ChatRoom> chatRooms;

    public User(String username) {
        this.username = username;
        this.chatRooms = new ArrayList<>();
    }

    public String getUsername() {
        return username;
    }

    public void addChatRoom(ChatRoom room) {
        chatRooms.add(room);
    }

    public void removeChatRoom(ChatRoom room) {
        chatRooms.remove(room);
    }

    public void sendMessage(ChatRoom room, String content) {
        if (chatRooms.contains(room)) {
            Message message = new Message(this, content);
            room.sendMessage(message);
        }
    }

    public void receiveMessage(Message message) {
        System.out.println(username + " received message: " + message.getContent() + " from " + message.getSender().getUsername());
    }

    public void receiveMessageConfirmation(Message message) {
        System.out.println(username + " sent message: " + message.getContent() + " at " + message.getTimestamp());
    }
}

class Message {
    private User sender;
    private String content;
    private LocalDateTime timestamp;

    public Message(User sender, String content) {
        this.sender = sender;
        this.content = content;
        this.timestamp = LocalDateTime.now();
    }

    public User getSender() {
        return sender;
    }

    public String getContent() {
        return content;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }
}

class ChatRoom {
    private String roomId;
    private List<User> users;
    private List<Message> messages;

    public ChatRoom(String roomId) {
        this.roomId = roomId;
        this.users = new ArrayList<>();
        this.messages = new ArrayList<>();
    }

    public String getRoomId() {
        return roomId;
    }

    public void addUser(User user) {
        users.add(user);
        user.addChatRoom(this);
    }

    public void removeUser(User user) {
        users.remove(user);
        user.removeChatRoom(this);
    }

    public void sendMessage(Message message) {
        messages.add(message);
        notifyUsers(message);
    }

    private void notifyUsers(Message message) {
        message.getSender().receiveMessageConfirmation(message);
        for (User user : users) {
            if (!user.equals(message.getSender())) {
                user.receiveMessage(message);
            }
        }
    }
}


class ChatApplication {
    private static ChatApplication instance;
    private Map<String, ChatRoom> chatRooms;
    
    private ChatApplication() {
        chatRooms = new HashMap<>();
    }

    public static synchronized ChatApplication getInstance() {
        if (instance == null) {
            instance = new ChatApplication();
        }
        return instance;
    }

    public ChatRoom createChatRoom(String roomId) {
        ChatRoom chatRoom = new ChatRoom(roomId);
        chatRooms.put(roomId, chatRoom);
        return chatRoom;
    }

    public ChatRoom joinChatRoom(String roomId, User user) {
        ChatRoom chatRoom = chatRooms.get(roomId);
        if (chatRoom != null) {
            chatRoom.addUser(user);
        }
        return chatRoom;
    }
}

public class Chat {
    public static void main(String[] args) {
        ChatApplication app = ChatApplication.getInstance();

        User user1 = new User("Alice");
        User user2 = new User("Bob");

        ChatRoom chatRoom1 = app.createChatRoom("room1");
        ChatRoom chatRoom2 = app.createChatRoom("room2");

        chatRoom1.addUser(user1);
        chatRoom1.addUser(user2);

        chatRoom2.addUser(user1);

        user1.sendMessage(chatRoom1, "Hello, Bob in room1!");
        user2.sendMessage(chatRoom1, "Hi, Alice in room1!");

        user1.sendMessage(chatRoom2, "Hello, anyone in room2!");
    }
}


