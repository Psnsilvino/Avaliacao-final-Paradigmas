# Avaliação final: Paradigmas

---

## Propósito: Entender a lógica por trás das interações entre usuários em redes sociais.

---

## Conceitos

#### Analisando o enunciado do propósito podemos perceber 3 núcleos-chave: __Interações__, __Redes Sociais__ e __Usuários__.

#### Começando pela parte mais importante do propósito: __Interações__. Para ocorrer uma interação em uma rede social são necessários uma ação, dois usuarios, um que realiza a ação e outro que a recebe, e, com a finalidade de medir o nível de interação, um valor numérico.

#### Uma rede social é um ambiente complexo que permite que os usuários se conectem, interajam e compartilhem informações e interesses através da internet. A fim de simplificar para o projeto podemos definir que uma rede social é uma composição de __Usuarios__ e __Postagens__. Na rede é possível registrar e remover um usuário, adicionar uma postagem, achar um usuário e achar um post, além dos métodos acessores necessários.

##### Uma postagem possui um identificador, um autor, o conteúdo da postagem, uma lista com os usuários que gostaram da postagem e outra lista que contém os __comentarios__ feitos naquela postagem. Na postagem, pode-se adicionar o usuário que curtiu a postagem na lista de likes e adicionar um comentário na lista de comentários, além dos métodos acessores necessários.

##### Um comentário possui um autor e seu conteúdo.

#### Um usuário é uma pessoa que criou uma conta em uma rede social. Nisso temos dois conceitos que se relacionam: __Conta__ e __Usuario__.

##### Uma conta possui quatro listas: uma com aqueles que um usuário em questão segue, uma com aqueles que seguem esse usuário, uma com as mensagens enviadas pelo usuários e outra que tem as mensagens enviadas ao usuário. A conta permite seguir e deixar de seguir outros usuários e enviar mensagens para amigos, além de conseguir acessar os parâmetros necessários por métodos. 

#####  Um usuário possui, além das listas advindas da conta, um identificador, um username, uma lista de interações realizadas por ele, e um hashmap com aqueles que o usuário interagiu e o valor somado de suas interações. Com o usuário é possível adicionar, remover e procurar interações, definir os níveis de interação baseado nas interações preexistentes e acessar o id e username com os métodos acessores. 

---

## Implementação

#### Começando a implementação com o usuario que alem de estar nos núcleos analisados na frase do propósito ele também compõe os outros dois núcleos.

Antes disso precisamos implementar a classe de mensagens.

### class Message:
```java
    public class Message {
        private User sender;
        private User receiver;
        private String content;

        public Message(User sender, User receiver, String content) {
            this.sender = sender;
            this.receiver = receiver;
            this.content = content;
        }
    }
```

### class Account:
```java
    import java.util.List;
    import java.util.ArrayList;

    public abstract class Account {
        protected List<User> follows;
        protected List<User> followers;
        protected List<Message> sentMessages;
        protected List<Message> receivedMessages;

        public Account(){
            this.follows = new ArrayList<>();
            this.followers = new ArrayList<>();
            this.sentMessages = new ArrayList<>();
            this.receivedMessages = new ArrayList<>();
        }
    public boolean isFollowing(User targetUser) {
        for (User user : follows) {
            if (user.equals(targetUser)) {
                return true;
            }
        }
        return false;
    }

    public void follow(User followingUser, User followedUser) {
        if (!isFollowing(followedUser)) {
            followingUser.follows.add(followedUser);
            followedUser.followers.add(followingUser);

            followingUser.addInteraction(Action.Follow, followedUser, 2);
        }
        else {
            System.out.println("Usuario ja seguido");
        }
    }

    public void unfollow(User unfollowingUser, User unfollowedUser) {
        if (isFollowing(unfollowedUser)) {
            unfollowingUser.follows.remove(unfollowedUser);
            unfollowedUser.followers.remove(unfollowingUser);

            Interaction unfollowInteraction = unfollowingUser.findInteraction(Action.Follow, unfollowedUser);
            unfollowingUser.removeInteraction(unfollowInteraction);
        }
        else {
            System.out.println("Usuario nao encontrado");
        }
    }

    public void sendMessage(User sender, User receiver, String content) {
        if (sender.isFollowing(receiver) && receiver.isFollowing(sender)) {
            Message newMessage = new Message(sender, receiver, content);
            sender.sentMessages.add(newMessage);
            receiver.receivedMessages.add(newMessage);

            sender.addInteraction(Action.Message, receiver, 3);
        }
        else {
            System.out.println("Nao é possivel enviar mensagem ao usuario");
        }
    }
}
```

### class User:
```java
    import java.util.ArrayList;
    import java.util.List;
    import java.util.HashMap;

    public class User extends Account{
        private int userID;
        private String username;
        private List<Interaction> interactions;
        private HashMap<User, Integer> interactionLevels;

        public User(int id, String username) {
            super();
            this.userID = id;
            this.username = username;
            this.interactions = new ArrayList<>();
            this.interactionLevels = new HashMap<User, Integer>();
        }

        public int getId() {
            return this.userID;
        }

        public String getUsername() {
            return this.username;
        }

        public void addInteraction(Action action, User otherUser, int value) {
            Interaction newInteraction = new Interaction(action, this, otherUser, value);
            this.interactions.add(newInteraction);
        }

        public void removeInteraction(Interaction interaction) {
            this.interactions.remove(interaction);
        }

        public Interaction findInteraction(Action action, User otherUser) {
            for (Interaction interaction : interactions) {
                if (interaction.getAction() == action && interaction.getUser1() == this && interaction.getUser2() == otherUser) {
                    return interaction;
                }
            }
            return null;
        }

        public void interactionLevels(){
            for (Interaction i : this.interactions) {
                if (!(this.interactionLevels.containsKey(i.getUser2()))) {
                    this.interactionLevels.put(i.getUser2(), i.getValue());
                }
                else {
                    int oldValue = this.interactionLevels.get(i.getUser2());
                    int newValue = i.getValue() + oldValue;
                    this.interactionLevels.replace(i.getUser2(), newValue);
                }
            }
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj) {
                return true;
            }

            if (obj == null || getClass() != obj.getClass()) {
                return false;
            }

            return false;
        }


    }   
```

Agora para implementar as interações só precisamos das ações. Sobre as ações podemos utilizar um enum:

### enum Action:
```java
    public enum Action {
        Message,
        Follow,
        Like,
        Comment;
    }
```

Com isso temos todos os requisitos para a classe de interações.

### class Interaction:
```java
    public class Interaction {

        private Action action;
        private User user1;
        private User user2;
        private int value;

        public Interaction(Action action, User user1, User user2, int value) {
            this.action = action;
            this.user1 = user1;
            this.user2 = user2;
            this.value = value;
        }

        public Action getAction() {
            return this.action;
        }

        public User getUser1() {
            return this.user1;
        }

        public User getUser2() {
            return this.user2;
        }

        public int getValue() {
            return this.value;
        }

    }
```

Para implementar o ultimo nucleo ainda falta implementar o codigo referente as postagens. E para as postagens falta o codigo dos comentarios.

### class Comment:
```java
    public class Comment {
        private User author;
        private String text;

        public Comment(User author, String text) {
            this.author = author;
            this.text = text;
        }
    }
```

### class Post:
```java
    import java.util.ArrayList;
    import java.util.List;

    public class Post {
        private int id;
        private User author;
        private String content;
        private List<User> likes;
        private List<Comment> comments;

        public Post(int id, User author, String content) {
            this.id = id;
            this.author = author;
            this.content = content;
            this.likes = new ArrayList<>();
            this.comments = new ArrayList<>();
        }

        public void addLike(User user) {
            likes.add(user);

            user.addInteraction(Action.Like, this.author, 1);
        }

        public void addComment(User user, String text) {
            Comment comment = new Comment(user, text);
            comments.add(comment);

            user.addInteraction(Action.Comment, this.author, 2);
        }

        public int getId() {
            return this.id;
        }

        public User getAuthor() {
            return this.author;
        }

        public String getContent() {
            return this.content;
        }
    }
```

### class Network
```java
    import java.util.ArrayList;
    import java.util.List;

    public class Network {
        private List<User> users;
        private List<Post> posts;

        public Network() {
            this.users = new ArrayList<>();
            this.posts = new ArrayList<>();
        }

        public void registerUser(User user) {
            this.users.add(user);
        }

        public void createPost(Post post) {
            this.posts.add(post);
        }

        public void removeUser(User user) {
            this.users.remove(user);
        }

        public int getUsersSize() {
            return this.users.size();
        }

        public int getPostsSize() {
            return this.posts.size();
        }

        public User findUser(String username) {
            User foundUser;
            for (User u : users) {
                if (u.getUsername().equals(username)) {
                    foundUser = u;
                    return foundUser;
                }
            }
            return null;
        }

        public Post findPost(int postId) {
            Post foundPost;
            for (Post p : posts) {
                if (p.getId() == postId) {
                    foundPost = p;
                    return foundPost;
                }
            }
            return null;
        }
    }
```

Esse foi meu projeto orientado a objetos.

---

### Agora implementando em outros paradigmas

### Prolog
```prolog
    :- dynamic follows/3.
    :- dynamic message/3.
    :- dynamic post/3.
    :- dynamic interaction/3.
    :- dynamic network/2.
    :- dynamic interactionLevel/2.

    % user(Username)
    user(cabelo).
    user(silencio).
    user(mascara).
    user(vasco).
    user(black).

    % action(Action, Value)
    action(like, 1).
    action(follow, 2).
    action(comment, 3).
    action(action(message, 5)).

    % follows(User1, User2)
    follows(cabelo, mascara).
    follows(mascara, cabelo).
    follows(silencio, vasco).
    follows(vasco, silencio).

    % message(Sender, Receiver, Content)
    message(vasco, silencio, "Fala ai").
    message(silencio, vasco, "...").
    message(cabelo, mascara, "Bom dia").

    % comment(Author, Content).
    comment(vasco, "Vasco").

    % post(Author, Content, Likes, Comments)
    post(vasco, "Mais uma derrota do vasco", [black], []).
    post(silencio, "...", [], [comment(vasco, "Vasco")]).

    % interaction(User1, User2, Action).
    interaction(cabelo, mascara, action(follow, 1)).
    interaction(mascara, cabelo, action(follow, 1)).
    interaction(silencio, vasco, action(follow, 1)).
    interaction(vasco, silencio, action(follow, 1)).
    interaction(cabelo, mascara, action(message, 5)).
    interaction(vasco, silencio, action(message, 5)).
    interaction(silencio, vasco, action(message, 5)).

    % network(Users, Posts)
    network([], []).

    addUser(User) :- user(User),
                     network(Users, Posts), 
                     append(Users, User, List),
                     retract(network(Users, Posts)),
                     assert(network(List, Posts)).
    
    addPost(Post) :- post(Post),
                     network(Users, Posts), 
                     append(Posts, Post, List),
                     retract(network(Users, Posts)),
                     assert(network(Users, List)).
    
    sendMessage(Sender, Receiver, Content) :- user(Sender),
                                              user(Receiver),
                                              follows(Sender, Receiver),
                                              follows(Receiver, Sender),
                                              assert(message(Sender, Receiver, Content)),
                                              assert(interaction(Sender, Receiver, action(message, 5))).
    
    addLike(Post, Liker) :- post(Post),
                            user(Liker),
                            post(Author, Content, Likes, Comments),
                            append(Likes, Liker, List),
                            retract(post(Author, Content, Likes, Comments)),
                            assert(post(Author, Content, List, Comments)),
                            assert(interaction(Liker, Author, action(like, 1))).
    
    follow(Follower, Followed) :- user(Follower),
                                  user(Followed),
                                  \+ follows(Follower, Followed),
                                  assert(follows(Follower, Followed)),
                                  assert(interaction(Follower, Followed, action(follow, 2))).
    
    unfollow(Unfollower, Unfollowed) :- user(Unfollower),
                                  user(Unfollowed),
                                  follows(Unfollower, Unfollowed),
                                  retract(follows(Unfollower, Unfollowed)),
                                  retract(interaction(Follower, Followed, action(follow, 2))).

    commentPost(Post, Comment) :- post(Post),
                                  post(Author, Content, Likes, Comments),
                                  Comment(Cauthor, _),
                                  append(Comments, Comment, List),
                                  retract(post(Author, Content, Likes, Comments)),
                                  assert(post(Author, Content, Likes, List)),
                                  assert(Cauthor, Author, action(comment, 3)).
```