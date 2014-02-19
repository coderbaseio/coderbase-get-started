---
title: Double Locking For Singleton Design Pattern
languages: Java
draft: false
summary: Double locking mechanism for a Thread safe Singleton.
---

### The what and why of Singleton
Singleton is a design pattern which guarantees a class only has one instance at any given time. The Singleton pattern is often used by Abstract Factory, Builder, Facade, and State objects. A common technique with Singleton is Lazy loading which initializes/allocates on the first time the Singleton is used, rather than at start. 

A simple Singleton with Lazy loading is shown below in Java.

    public class GameState {
        private static GameState instance = null;

        private GameState() {}

        public static GameState getInstance() {
            if (instance == null) {
                instance = new GameState();
            }

            return instance;
        }
    }


### Thread Safety
The code above works perfectly fine, however it is not thread safe. When working in a multithreaded environment, it is important to ensure thread safety. There are multiple ways to do this and I highlight them below.


    public class GameState {
        private static volatile GameState instance = null;

        private GameState() {}

        public static GameState getInstance() {
            if (instance == null) {
                synchronized (GameState.class) {
                    instance = new GameState();  
                }
            }

            return instance;
        }
    }

In this example, we obtain a lock on the class as a whole before allowing construction of the GameState instance. 


    public class GameState {
        private static volatile GameState instance = null;

        private GameState() {}

        public static synchronized GameState getInstance() {
            if (instance == null) {
                instance = new GameState();
            }

            return instance;
        }
    }


This is a bit cleaner and much simplier, but reduces the amounts of concurrency in a multithreaded envirnoment.


    public class GameState {
        private static volatile GameState instance = null;
        private static Object _lock = new Object();

        private GameState() {}

        public static GameState getInstance() {
            if (instance == null) {
                synchronized(_lock) {
                    if (instance == null) {
                        instance = new GameState();
                    }
                }
            }

            return instance;
        }
    } 
This is the double locking pattern in which we use a private Object to obtain the lock on and before we attempt to lock the object, we check the locking criterion without actually acquiring the lock. 


### Why did I write this?

I wrote this because thread safety is not something I usually think about when I write my code, however it is very important and can lead to serrious issues if not designed well. I found Double Locking to be particuarly interesting because a) It dealt with design patterns and b) There were so many articles about what the best approach to this is.

*Disclaimer:* There are a few articles which state that double locking does not always work or is not the absolute best. It is a very interesting topic, but not suitable for a blog post.
