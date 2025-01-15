https://refactoring.guru/design-patterns

## Builder pattern
```java
public class User {
    private String name;
    private String gender;
    private int age;
    private String userId;
    private String ip;
    private String birthday;

    public User(String name, String gender, int age,
                String userId, String ip, String birthday) {
        this.name = name;
        this.gender = gender;
        this.age = age;
        this.userId = userId;
        this.ip = ip;
        this.birthday = birthday;
    }

    // problem 1:
    // need to overload many constructors
    // if not every attribute is required
    public User(String name, String gender, int age) {
        this(name, gender, age, '', '', '');
    }

    // problem 2:
    // not enough constructor to overload
    public User(String name){...}

    public User(String gender){...}  // error
}

// builder pattern
public class UserBuilder {
    private String name;
    private String gender;
    private int age;
    private String userId;
    private String ip;
    private String birthday;

    public UserBuilder setName(String name) {
        this.name = name;
        return this;
    }

    public UserBuilder setGender(String gender) {
        this.gender = gender;
        return this;
    }

    public UserBuilder setAge(int age) {
        this.age = age;
        return this;
    }

    public UserBuilder setUserId(String userId) {
        this.userId = userId;
        return this;
    }

    public UserBuilder setIp(String ip) {
        this.ip = ip;
        return this;
    }

    public UserBuilder setBirthday(String birthday) {
        this.birthday = birthday;
        return this;
    }

    public User build() {
        return new User(name, gender, age, 
                userId, ip, birthday);
    }
}

// to create a user object:
User curry = new UserBuilder()
        .setName("Curry")
        .setAge(30)
        .setGender("Male")
        .build();
```