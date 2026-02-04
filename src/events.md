# Events

Bevy uses a double-buffered event system that enables efficient event production and consumption with zero-allocation event consumers.

## Complete Example

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

fn event_producer(mut my_events: ResMut<Events<MyEvent>>) {
    my_events.send(MyEvent { message: "Hello".to_string() });
}

#[derive(Default)]
struct State {
    reader: EventReader<MyEvent>,
}

fn event_consumer(mut state: Local<State>, my_events: Res<Events<MyEvent>>) {
    for event in state.reader.iter(&my_events) {
        println!("received message: {}", event.message);
    }
}
```

## How It Works

`app.add_event::<MyEvent>()` adds a new Events resource for MyEvent and a system that swaps the Events<MyEvent> buffers every update. EventReaders are very cheap to create - they are essentially just an array index that tracks the last event that has been read.

## Built-in Events

Events are used in Bevy for features like:
- Window resizing
- Assets
- Input

## Trade-offs

The tradeoff for being both allocation and cpu efficient is that each system only has one chance to receive an event, otherwise it will be lost on the next update. This is the correct tradeoff for apps that run in a loop (ex: games).

