
# Bikeshedding `cell`

## Paper
http://wg21.link/P0561

## Usage

Seeing usage is helpful when thinking about names, so I have copied the example
from the paper. I added one example that relies on class type deduction, which I
expect to be common in C++17 code.

```c++
class Server {
 public:

  void SetConfig(Config new_config) {
    config_.update(std::make_unique<const Config>(std::move(new_config)));
  }

  void HandleRequest() {
    snapshot_ptr<const Config> config = config_.get_snapshot();
    // or, with class type deduction:
    snapshot_ptr config = config_.get_snapshot();

    // Use `config` like a unique_ptr<const Config>

  }

 private:
  cell<Config> config_;
};
```

## `snapshot_ptr`, Guard or Pointer?

Is `snapshot_ptr` a smart pointer or a scoped RAII adapter? And what about
thread affinity? sharing?

http://lists.isocpp.org/lib-ext/2017/11/5461.php:

Peter Dimov:
> As I said, if one thinks of creating a snapshot as obtaining a read lock
> (or calling rcu_read_lock), and destroying the snapshot as releasing the
> read lock (or calling rcu_read_unlock), the programming model is
> straightforward.
>
> The only problem we have (if any) is a naming one. Call it cell_read_lock
> instead of snapshot_ptr and nobody will think of passing it across threads.

Geoffrey Romer:
> ...but also nobody will think of dereferencing it. I take your point that
> naming could help solve this problem, but if so the name needs to somehow
> suggest both the "lock" aspect and the "pointer" aspect.

http://lists.isocpp.org/lib-ext/2017/11/5482.php:

Tony Van Eerd:
> ... PS 'snapshot' doesn't convey sharing. Which is fine for the const T,
> but not so fine when T is not const. (I know that's the lesser used version
> of the API, but worth keeping in mind)

### Tony Table

||atomic<br>_shared<br>_ptr|shared<br>_ptr|unique<br>_ptr|snapshot<br>_ptr|unique<br>_lock|lock<br>_guard|unique<br>_resource|cell|
| --------------| ---:| ---:| ---:| ---:| ---:| ---:| ---:| ---:|
| moveable      |   X |   O |   O |**O**|   O |   X |   O |   X |
| copyable      |   X |   O |   X |**X**|   X |   X |   X |   X |
| get() -> T*   |   O |   O |   O |**O**|   X |   X |   X |   X |
| get() -> T&   |   X |   X |   X |**X**|   X |   X |   O |   X |
| shared T      |   O |   O |   X |**O**|   O |   O |   X |   O |
| thread safe   |   O |   X |   X |**X**|   X |   X |   X |   O |
| thread affine |   X |   X |   X |**O**|   O |   O |   X |   X |

http://wg21.link/thread.req.lockable.basic#3
<br>http://wg21.link/thread.lock.guard
<br>http://wg21.link/thread.lock.unique
<br>http://wg21.link/P0052 (unique_resource)

## Name Wall

### Synchronization Primitives
Establish a new term of art.
- `cell`
- `reclaimatron` (exposition only)

### Containers
- succession
- progression
- state
- versioned_resource

### Snapshot containers
- snapshot_publisher
- snapshot_source
- snapshot_state

### Kinds of snapshots
- current_snapshot
- latest_snapshot

### Adjectives
- versioned
- reclaimed
- latest
- current
- snappable
- atomic (example adjectival class template name)
