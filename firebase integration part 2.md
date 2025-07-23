## 1. Create a Firebase Project

1. Go to the [Firebase Console](https://console.firebase.google.com/).
2. Click **“Add project”**, give it a name, and follow the prompts.
3. In your new project, under **Project settings → Your apps**, click the **</>** (Web) icon.
4. Register your app and copy the Firebase config object you’re shown (you’ll need it in Step 3).

## 2. Install the Firebase SDK

```bash
npm install firebase
# or
yarn add firebase
```

## 3. Store Your Firebase Config Securely

Add a `.env.local` (CRA) or `vite.env` (Vite) with your config keys:

```
REACT_APP_FIREBASE_API_KEY=…
REACT_APP_FIREBASE_AUTH_DOMAIN=…
REACT_APP_FIREBASE_PROJECT_ID=…
REACT_APP_FIREBASE_STORAGE_BUCKET=…
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=…
REACT_APP_FIREBASE_APP_ID=…
```

> *(In Vite, prefix with `VITE_` instead of `REACT_APP_`.)*

## 4. Initialize Firebase

```js
// src/firebase.js
import { initializeApp }   from 'firebase/app';
import { getFirestore }    from 'firebase/firestore';

const firebaseConfig = {
  apiKey:             process.env.REACT_APP_FIREBASE_API_KEY,
  authDomain:         process.env.REACT_APP_FIREBASE_AUTH_DOMAIN,
  projectId:          process.env.REACT_APP_FIREBASE_PROJECT_ID,
  storageBucket:      process.env.REACT_APP_FIREBASE_STORAGE_BUCKET,
  messagingSenderId:  process.env.REACT_APP_FIREBASE_MESSAGING_SENDER_ID,
  appId:              process.env.REACT_APP_FIREBASE_APP_ID,
};

const app       = initializeApp(firebaseConfig);
export const db = getFirestore(app);
```

---

## 5. Build a Products CRUD

### 5.1. Firestore Service

Create helper functions in `src/services/productService.js`:

```js
// src/services/productService.js
import {
  collection,
  addDoc,
  getDocs,
  doc,
  updateDoc,
  deleteDoc,
} from 'firebase/firestore';
import { db } from '../firebase';

const productsCol = collection(db, 'products');

export async function createProduct(data) {
  // data: { name, description, price, details }
  const ref = await addDoc(productsCol, {
    ...data,
    createdAt: Date.now()
  });
  return ref.id;
}

export async function getAllProducts() {
  const snapshot = await getDocs(productsCol);
  return snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
}

export async function updateProduct(id, data) {
  const ref = doc(db, 'products', id);
  await updateDoc(ref, data);
}

export async function deleteProduct(id) {
  const ref = doc(db, 'products', id);
  await deleteDoc(ref);
}
```

---

### 5.2. ProductForm Component (Create & Update)

```jsx
// src/components/ProductForm.jsx
import { useState, useEffect } from 'react';
import { createProduct, updateProduct } from '../services/productService';

export function ProductForm({ existing, onSaved }) {
  // existing = { id, name, description, price, details } or null
  const [form, setForm] = useState({
    name: '',
    description: '',
    price: '',
    details: ''
  });

  useEffect(() => {
    if (existing) setForm(existing);
  }, [existing]);

  const handleChange = e =>
    setForm(f => ({ ...f, [e.target.name]: e.target.value }));

  const handleSubmit = async e => {
    e.preventDefault();
    if (existing) {
      await updateProduct(existing.id, form);
    } else {
      await createProduct(form);
    }
    onSaved();
    setForm({ name: '', description: '', price: '', details: '' });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        placeholder="Name"
        value={form.name}
        onChange={handleChange}
        required
      />
      <textarea
        name="description"
        placeholder="Description"
        value={form.description}
        onChange={handleChange}
      />
      <input
        name="price"
        type="number"
        placeholder="Price"
        value={form.price}
        onChange={handleChange}
        required
      />
      <textarea
        name="details"
        placeholder="Details"
        value={form.details}
        onChange={handleChange}
      />
      <button type="submit">
        {existing ? 'Update' : 'Create'}
      </button>
    </form>
  );
}
```

---

### 5.3. ProductList Component (Read & Delete)

```jsx
// src/components/ProductList.jsx
import { useEffect, useState } from 'react';
import { getAllProducts, deleteProduct } from '../services/productService';
import { ProductForm } from './ProductForm';

export function ProductList() {
  const [products, setProducts] = useState([]);
  const [editing, setEditing] = useState(null);

  const load = async () => {
    const items = await getAllProducts();
    setProducts(items);
    setEditing(null);
  };

  useEffect(() => {
    load();
  }, []);

  return (
    <div>
      <h2>{editing ? 'Edit Product' : 'New Product'}</h2>
      <ProductForm existing={editing} onSaved={load} />

      <h2>All Products</h2>
      <ul>
        {products.map(p => (
          <li key={p.id}>
            <strong>{p.name}</strong> — ${p.price}
            <button onClick={() => setEditing(p)}>Edit</button>
            <button
              onClick={async () => {
                await deleteProduct(p.id);
                load();
              }}
            >
              Delete
            </button>
            <p>{p.description}</p>
            <small>{p.details}</small>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### 5.4. Wire It Up

In your `App.jsx` (or wherever):

```jsx
import React from 'react';
import { ProductList } from './components/ProductList';

function App() {
  return (
    <div style={{ padding: 20 }}>
      <h1>Products Manager</h1>
      <ProductList />
    </div>
  );
}

export default App;
```

---

You now have a working Firebase‑backed React app with full Create, Read, Update, and Delete for products!
