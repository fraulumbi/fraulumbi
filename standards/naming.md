# Naming

- Names describe **what a thing is or does**, never how it's implemented.
  `fetch_active_orders` > `get_data2`. `retry_count` > `n`.
- Follow the host language's casing without exception: `snake_case` for Elixir,
  Python, and SQL identifiers; `camelCase` for JS/TS locals; `PascalCase` for
  modules, types, and classes; `SCREAMING_SNAKE_CASE` for compile-time constants.
- Booleans read as an assertion: `active?`, `is_expired`, `has_license`. Never
  negate in the name (`not_disabled` forces a double negative at every call site).
- Functions that perform side effects use verbs (`sync_inventory`, `send_receipt`).
  Functions that only compute use nouns or predicates (`total_weight`, `valid?`).
- Elixir: a trailing `!` means "raises on failure", a trailing `?` means "returns a
  boolean". Don't use either for anything else.
- Abbreviate only what the domain already abbreviates (`id`, `url`, `sku`, `uom`).
  Spell out everything else.
- No unexplained numbers or strings in logic. Bind them to a named constant or
  module attribute so the meaning lives next to the value.
