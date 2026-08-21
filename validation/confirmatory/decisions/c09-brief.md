# Decision C09 - preventing overselling inventory

Under concurrent checkouts, code does: `SELECT stock FROM items WHERE id=$1;`
(app checks stock > 0) then `UPDATE items SET stock = stock - 1 WHERE id=$1;` in
the default READ COMMITTED isolation. To fix a suspected race, a dev proposes
raising the transaction isolation to **REPEATABLE READ**. **Question:** does
switching to REPEATABLE READ make this oversell-safe? Commit and name the single
biggest risk.
