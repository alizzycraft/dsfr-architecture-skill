# Examples

Use these examples as placement guides, not as mandatory syntax templates.

## Example 1: Single Domain With Derived Pricing

```ts
type CartItem = { id: string; price: number; quantity: number };

const calculateSubtotal = (items: CartItem[]): number =>
  items.reduce((sum, item) => sum + item.price * item.quantity, 0);

const applyDiscount = (subtotal: number): number =>
  subtotal > 100 ? subtotal * 0.9 : subtotal;

const calculateTax = (amount: number): number => amount * 0.15;

class CartPricingDomain {
  readonly items = signal<CartItem[]>([]);
  readonly subtotal = computed(() => applyDiscount(calculateSubtotal(this.items())));
  readonly total = computed(() => this.subtotal() + calculateTax(this.subtotal()));

  addItem(item: CartItem) {
    this.items.update(items => [...items, item]);
  }
}
```

Placement:

- `items` is source state.
- `subtotal` and `total` are derived state.
- pricing math is pure transformation logic.
- `addItem` is an entry point.

## Example 2: Async IO Routed Through Effect

```ts
type UserProfile = { id: string; name: string };

const normalizeProfile = (raw: ApiUser): UserProfile => ({
  id: raw.id,
  name: raw.display_name,
});

class ProfileDomain {
  readonly profile = signal<UserProfile | null>(null);
  readonly loading = signal(false);

  startLoading() {
    this.loading.set(true);
  }

  receiveProfile(profile: UserProfile) {
    this.profile.set(profile);
    this.loading.set(false);
  }

  failLoading() {
    this.loading.set(false);
  }
}

async function loadProfileEffect(
  api: ProfileApi,
  domain: ProfileDomain,
  userId: string,
) {
  domain.startLoading();
  try {
    const raw = await api.fetchProfile(userId);
    domain.receiveProfile(normalizeProfile(raw));
  } catch {
    domain.failLoading();
  }
}
```

Placement:

- IO is isolated in the effect.
- transport mapping is allowed in the effect.
- domain state changes only through entry points.
- no business logic is embedded in the effect.

## Example 3: Cross-Domain Coordination Through Orchestrator

```ts
class InventoryDomain {
  reserveItem(itemId: string, quantity: number) {}
}

class CartDomain {
  confirmReservation(itemId: string, quantity: number) {}
}

class CartCheckoutOrchestrator {
  constructor(
    private readonly inventory: InventoryDomain,
    private readonly cart: CartDomain,
  ) {}

  reserveCartItem(itemId: string, quantity: number) {
    this.inventory.reserveItem(itemId, quantity);
    this.cart.confirmReservation(itemId, quantity);
  }
}
```

Placement:

- domains do not mutate one another directly.
- cross-domain flow is explicit.
- the orchestrator coordinates; ownership stays local.
