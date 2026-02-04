# Events

Bevy uses an efficient event system for communication between systems.

## Complete Example (Bevy 0.5+)

Here is a complete Bevy app that produces and consumes a custom event:

```rust
fn main() {
    App::build()
        .add_event::<MyEvent>()
        .add_system(event_producer.system())
        .add_system(event_consumer.system())
        .run();
}

struct MyEvent {
    message: String,
}

fn event_producer(mut my_events: EventWriter<MyEvent>) {
    my_events.send(MyEvent { 
        message: "Hello".to_string() 
    });
}

fn event_consumer(mut my_events: EventReader<MyEvent>) {
    for event in my_events.iter() {
        println!("received message: {}", event.message);
    }
}
```

## Simplified API (Bevy 0.5)

**Changed in Bevy 0.5**

The event API was significantly simplified:

**Before (0.4):**
```rust
fn event_consumer(
    mut state: Local<EventReader<MyEvent>>,
    my_events: Res<Events<MyEvent>>
) {
    for event in state.reader.iter(&my_events) {
        println!("received: {}", event.message);
    }
}

fn event_producer(mut my_events: ResMut<Events<MyEvent>>) {
    my_events.send(MyEvent { message: "Hello".to_string() });
}
```

**After (0.5):**
```rust
fn event_consumer(mut my_events: EventReader<MyEvent>) {
    for event in my_events.iter() {
        println!("received: {}", event.message);
    }
}

fn event_producer(mut my_events: EventWriter<MyEvent>) {
    my_events.send(MyEvent { message: "Hello".to_string() });
}
```

**Benefits:**
- Single parameter instead of two for reading events
- `EventWriter` for producing events (clearer intent)
- Automatic state management (no Local<EventReader> needed)

## How It Works

`app.add_event::<MyEvent>()` adds a new Events resource for MyEvent and a system that swaps the Events<MyEvent> buffers every update. EventReaders are very cheap to create - they are essentially just an array index that tracks the last event that has been read.

## Built-in Events

Events are used in Bevy for features like:
- Window resizing
- Assets
- Input

## Trade-offs

The tradeoff for being both allocation and cpu efficient is that each system only has one chance to receive an event, otherwise it will be lost on the next update. This is the correct tradeoff for apps that run in a loop (ex: games).

