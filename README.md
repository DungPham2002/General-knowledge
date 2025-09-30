# General-knowledge

Section 1: Core React Concep

Question 1.1:
useState is used to manage values that affect the UI. When the state changes, the component re-renders. useRef stores values across renders but changing it won’t trigger a re-render. It’s often used to keep a DOM reference or temporary data. So, if I need the UI to update, I use useState; if I just need to keep a value silently, I use useRef.

Question 1.2
//Missing import useState
import React, { useState } from 'react';

function UserList({ users }) {
  const [selectedUser, setSelectedUser] = useState();

  return (
    <div>
      <h2>Users</h2>
      {users.map((user) => (
        <div onClick={() => setSelectedUser(user.id)}>  //must set state to user instead of user.id
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}

      {selectedUser && (
        <div>
          <h3>Selected: {selectedUser.name}</h3>
        </div>
      )}
    </div>
  );
}

Question 1.3
import { useState, useCallback } from 'react';

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue(prev => !prev);
  }, []);

  const reset = useCallback(() => {
    setValue(initialValue);
  }, [initialValue]);

  return [value, toggle, reset];
}


Section 2: State Management & Effects
import React, { useEffect, useState } from "react";

interface User {
  id: number;
  name: string;
  username: string;
  email: string;
  address: {
    street: string;
    suite: string;
    city: string;
    zipcode: string;
    geo: {
      lat: number;
      lng: number;
    }
    phone: number;
    website: string;
    company: {
      name: string;
      catchPhrase: string;
      bs: string
    }
  }
}

function UserProfile() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);
        const res = await fetch("https://jsonplaceholder.typicode.com/users/1", {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error("Failed to fetch user");
        const data = await res.json();
        setUser(data);
      } catch (err: any) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchUser();

    return () => {
      controller.abort();
    };
  }, []);

  const handleNameChange = async (newName: string) => {
    if (!user) return;

    const prevUser = { ...user };
       setUser({ ...user, name: newName });

    try {
      const res = await fetch(`https://jsonplaceholder.typicode.com/users/${user.id}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ ...user, name: newName }),
      });
      if (!res.ok) throw new Error("Failed to update");
    } catch (err: any) {
      setError(err.message);
      setUser(prevUser); // rollback on error
    }
  };

  if (loading) return <p>Loading user...</p>;
  if (error) return <p style={{ color: "red" }}>Error: {error}</p>;

  return (
    <div>
      {user && (
        <div>
          <input
            type="text"
            value={user.name}
            onChange={(e) => handleNameChange(e.target.value)}
          />
        </div>
      )}
    </div>
  );
}

Section 3: Component Design & Props
import React, { useEffect, useRef } from "react";
import ReactDOM from "react-dom";

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
  closeButton?: React.ReactNode;
}

const Modal: React.FC<ModalProps> = ({ isOpen, onClose, children, closeButton }) => {
  const modalRef = useRef<HTMLDivElement | null>(null);

  // prevent body scroll
  useEffect(() => {
    if (isOpen) document.body.style.overflow = "hidden";
    else document.body.style.overflow = "";
    return () => {
      document.body.style.overflow = "";
    };
  }, [isOpen]);

  // focus modal when open
  useEffect(() => {
    if (isOpen && modalRef.current) {
      modalRef.current.focus();
    }
  }, [isOpen]);

  // close on ESC
  useEffect(() => {
    const handleEsc = (e: KeyboardEvent) => {
      if (e.key === "Escape") onClose();
    };
    if (isOpen) document.addEventListener("keydown", handleEsc);
    return () => document.removeEventListener("keydown", handleEsc);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return ReactDOM.createPortal(
    <div
      className="fixed inset-0 z-[1000] flex items-center justify-center"
      aria-modal="true"
      role="dialog"
    >
      {/* Backdrop */}
      <div
        onClick={onClose}
        className="absolute inset-0 bg-black/50"
      />

      {/* Content */}
      <div
        ref={modalRef}
        tabIndex={-1}
        className="relative z-[1001] min-w-[300px] rounded-lg bg-white p-5 shadow-lg focus:outline-none"
      >
        {/* Close button */}
        <div className="absolute right-3 top-3">
          {closeButton ? (
            <span onClick={onClose} className="cursor-pointer">
              {closeButton}
            </span>
          ) : (
            <button
              onClick={onClose}
              aria-label="Close modal"
              className="rounded-full p-1 hover:bg-gray-100"
            >
              ✕
            </button>
          )}
        </div>

        {children}
      </div>
    </div>,
    document.body
  );
};


Section 4: Code Review
function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [filter, setFilter] = useState('');
  // Need check error
  // const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/products')
      .then((res) => res.json())
      .then((data) => {
        setProducts(data);
      })
      // Need check error
      // .catch((err) => setError(err.message))  
      // .finally(() => setLoading(false));
  }, []);

  const filteredProducts = products.filter((p) => p.name.toLowerCase().includes(filter.toLowerCase())) || []; //Add fallback
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return (
    <div>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} placeholder="Search products..." />

      //Check if no product found
      {filteredProducts.length > 0 ? (
        <div>
          {filteredProducts.map((product) => (
            <div
              key={product.id}
              style={{ border: "1px solid #ccc", margin: "10px", padding: "10px" }}
            >
              <h3>{product.name}</h3>
              <p>${product.price}</p>
              <img src={product.image} alt={product.name} style={{ width: "100px" }} />
            </div>
          ))}
        </div>
      ) : (
        <div>No products found.</div>
      )}
    </div>
  );
}
