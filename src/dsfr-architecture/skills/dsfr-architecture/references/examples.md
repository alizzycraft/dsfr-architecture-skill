# Examples

Use these examples as placement guides, not as mandatory syntax templates.

## Example 1: Single Domain With Derived Pricing

```ts
type CartItem = { id: string; price: number; quantity: number };

class CartPricingDomain {
  readonly items = signal<CartItem[]>([]);
  readonly subtotal = computed(() =>
    CartPricingDomain.applyDiscount(
      CartPricingDomain.calculateSubtotal(this.items()),
    ),
  );
  readonly total = computed(() =>
    this.subtotal() + CartPricingDomain.calculateTax(this.subtotal()),
  );

  private static calculateSubtotal(items: CartItem[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  private static applyDiscount(subtotal: number): number {
    return subtotal > 100 ? subtotal * 0.9 : subtotal;
  }

  private static calculateTax(amount: number): number {
    return amount * 0.15;
  }

  addItem(item: CartItem) {
    this.items.update(items => [...items, item]);
  }
}
```

Placement:

- `items` is source state.
- `subtotal` and `total` are derived state.
- pricing math is pure transformation logic co-located inside the domain class.
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

## Example 4: Optimistic Update With Rollback

```ts
type CartSnapshot = { items: CartItem[] };

class CartDomain {
  readonly items = signal<CartItem[]>([]);
  readonly pendingSnapshot = signal<CartSnapshot | null>(null);

  applyQuantityChange(itemId: string, quantity: number) {
    this.pendingSnapshot.set({ items: this.items() });
    this.items.update(items =>
      items.map(item => item.id === itemId ? { ...item, quantity } : item),
    );
  }

  confirmQuantityChange() {
    this.pendingSnapshot.set(null);
  }

  rollbackQuantityChange() {
    const snapshot = this.pendingSnapshot();
    if (!snapshot) return;
    this.items.set(snapshot.items);
    this.pendingSnapshot.set(null);
  }
}

async function updateCartQuantityEffect(
  api: CartApi,
  domain: CartDomain,
  itemId: string,
  quantity: number,
) {
  domain.applyQuantityChange(itemId, quantity);
  try {
    await api.updateQuantity(itemId, quantity);
    domain.confirmQuantityChange();
  } catch {
    domain.rollbackQuantityChange();
  }
}
```

Placement:

- optimistic mutation starts through an intent entry point
- success and rollback are handled through result entry points
- the effect does IO only and routes outcomes back into the domain

## Example 5: Read-Only Composition Layer

```ts
class AuthDomain {
  currentUser() {
    return this.user();
  }
}

class PricingDomain {
  currentPrice() {
    return this.price();
  }
}

class CheckoutReadModel {
  constructor(
    private readonly auth: AuthDomain,
    private readonly pricing: PricingDomain,
  ) {}

  summary() {
    return {
      user: this.auth.currentUser(),
      total: this.pricing.currentPrice(),
    };
  }
}
```

Placement:

- the read model owns no mutable state
- it composes read-only views across domains
- it does not replace orchestration for stateful behavior
