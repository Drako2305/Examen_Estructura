# Urban Courier Data Structures Exam

A TypeScript/Vite single-page application implementing the courier case study in English.

## Run

```bash
npm install
npm run dev
```

For a production check, run `npm run build`.

## Design decisions

- `DoublyLinkedList` models the editable route and exposes forward/backward traversal.
- `Queue` models pending and next-day retry shipments. Enqueue and dequeue are O(1).
- `Stack` models the motorcycle trunk. Push and pop are O(1).
- The UI only calls public `CourierSystem` methods and renders returned snapshots; it never edits a structure directly.
- Loading an occupied trunk is rejected and must be explicitly unloaded first.
- Failed deliveries use an auxiliary stack, count every move, preserve the relative order of the other packages, and move the failed stop to the route tail.

Use **Load sample data** to exercise the complete flow.
