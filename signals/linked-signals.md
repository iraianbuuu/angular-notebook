# Linked Signals

## Table of Contents

- [`linkedSignal`](#linkedsignal)
- [Options](#options)

## `linkedSignal`

`linkedSignal` is a utility that allows us to create a signal that is linked to another signal. It behaves like `computed` signals with a writable side.

`linkedSignal` is like `computed` but with 2 main differences :

1. The derived value of `linkedSignal` can be overridden at any time.

2. The `linkedSignal` provide access to the previous values

### Options

- `source` : The source signal that the linked signal is linked to.
- `computation` : It is a function that receives the new value of source and a previous object. The previous object contains the previous value of source and the previous value of the computation.
- `equal` : It is a function that compares the new value of source and the previous value of source.

```ts
import { Component, linkedSignal, signal } from "@angular/core";

@Component({
  selector: "app-linked-signals",
  imports: [],
  template: `
    <ul>
      @for (reaction of reactions(); track $index) {
      <li
        [class.selected]="isSelected(reaction)"
        (click)="selectedReaction.set(reaction)"
      >
        {{ reaction }}
      </li>
      }
    </ul>
    <span>Selected Reaction: {{ selectedReaction() }}</span>
    <button class="btn btn-primary" (click)="addReaction()">
      More Reaction
    </button>
    <button class="btn btn-primary" (click)="removeReaction()">
      Less Reaction
    </button>
  `,
  styles: ``,
})
export class LinkedSignalsComponent {
  reactions = signal<string[]>(["🤩", "😒", "🤔"]);
  // selectedReaction = linkedSignal(() => this.reactions()[-1]);

  selectedReaction = linkedSignal<string[], string>({
    source: () => this.reactions(),
    computation: (source, previous) => {
      return (
        source.find((reaction) => reaction === previous?.value) ?? source[0]
      );
    },
  });

  addReaction = () => {
    this.reactions.set(["🤣", "😊", "😎", "😯", "🫠"]);
  };

  removeReaction = () => {
    this.reactions.set(["🤩", "😒", "🤔"]);
  };

  isSelected = (reaction: string) => {
    return this.selectedReaction() === reaction;
  };
}
```
