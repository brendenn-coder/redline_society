/*
REDLINE SOCIETY — LIVE DEPLOYABLE STORE (NEXT.JS 14 APP ROUTER)
STACK:
- Next.js 14 (App Router)
- TailwindCSS
- Stripe Checkout (Live Payments)
- Supabase (optional DB)
- Vercel Deployment Ready
*/

/*****************************************************************
📁 PROJECT STRUCTURE
*****************************************************************

/app
  layout.tsx
  page.tsx (Home)
  shop/page.tsx
  product/[id]/page.tsx
  checkout/page.tsx
  success/page.tsx

/app/api/stripe/route.ts
/lib/products.ts
/lib/stripe.ts
/lib/cart.ts (optional)

/components/Navbar.tsx
/components/ProductCard.tsx
/components/CartDrawer.tsx

/styles/globals.css

*****************************************************************
📦 PRODUCT DATABASE (lib/products.ts)
*****************************************************************/

export const products = [
  {
    id: "tee-matte-black",
    name: "Matte Black RS Tee",
    price: 5000,
    description: "Flagship heavyweight RS tee with crimson accents.",
    collection: "Premium",
    images: ["/products/tee-black.jpg"],
  },
  {
    id: "hoodie-washed-black",
    name: "Washed Black RS Hoodie",
    price: 9500,
    description: "Oversized washed hoodie with racing back print.",
    collection: "Racing",
    images: ["/products/hoodie-washed.jpg"],
  },
  {
    id: "tee-navy",
    name: "Midnight Navy RS Tee",
    price: 5500,
    description: "Clean premium navy RS essential.",
    collection: "Premium",
    images: ["/products/tee-navy.jpg"],
  },
];

/*****************************************************************
🧠 STRIPE CONFIG (lib/stripe.ts)
*****************************************************************/

import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-06-20",
});

/*****************************************************************
💳 STRIPE CHECKOUT API (app/api/stripe/route.ts)
*****************************************************************/

import { stripe } from "@/lib/stripe";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const { cart } = await req.json();

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ["card"],
    mode: "payment",
    line_items: cart.map((item: any) => ({
      price_data: {
        currency: "usd",
        product_data: {
          name: item.name,
        },
        unit_amount: item.price,
      },
      quantity: 1,
    })),
    success_url: `${process.env.NEXT_PUBLIC_URL}/success`,
    cancel_url: `${process.env.NEXT_PUBLIC_URL}/checkout`,
  });

  return NextResponse.json({ url: session.url });
}

/*****************************************************************
🏠 HOME PAGE (app/page.tsx)
*****************************************************************/

import Link from "next/link";

export default function Home() {
  return (
    <main className="flex flex-col items-center justify-center h-screen bg-black text-white">
      <h1 className="text-6xl font-bold tracking-widest">REDLINE SOCIETY</h1>
      <p className="text-white/60 mt-4">Engineered for Motion</p>

      <Link href="/shop" className="mt-10 px-6 py-3 bg-white text-black">
        Enter Shop
      </Link>
    </main>
  );
}

/*****************************************************************
🛍️ SHOP PAGE (app/shop/page.tsx)
*****************************************************************/

import { products } from "@/lib/products";
import Link from "next/link";

export default function Shop() {
  return (
    <div className="p-10 bg-black text-white min-h-screen">
      <h2 className="text-3xl mb-8 tracking-widest">SHOP DROP 01</h2>

      <div className="grid md:grid-cols-3 gap-6">
        {products.map((p) => (
          <div key={p.id} className="border border-white/10 p-4">
            <h3>{p.name}</h3>
            <p className="text-white/60">${p.price / 100}</p>

            <Link href={`/product/${p.id}`} className="text-sm underline">
              View Product
            </Link>
          </div>
        ))}
      </div>
    </div>
  );
}

/*****************************************************************
📦 PRODUCT PAGE (app/product/[id]/page.tsx)
*****************************************************************/

import { products } from "@/lib/products";
import AddToCart from "@/components/ProductCard";

export default function ProductPage({ params }: any) {
  const product = products.find((p) => p.id === params.id);

  if (!product) return <div>Not found</div>;

  return (
    <div className="p-10 bg-black text-white min-h-screen">
      <h1 className="text-4xl">{product.name}</h1>
      <p className="text-white/60 mt-4">{product.description}</p>

      <p className="mt-4">${product.price / 100}</p>

      <AddToCart product={product} />
    </div>
  );
}

/*****************************************************************
🛒 CHECKOUT PAGE (app/checkout/page.tsx)
*****************************************************************/

"use client";

import { useState } from "react";

export default function Checkout() {
  const [cart, setCart] = useState([]);

  const checkout = async () => {
    const res = await fetch("/api/stripe", {
      method: "POST",
      body: JSON.stringify({ cart }),
    });

    const data = await res.json();
    window.location.href = data.url;
  };

  return (
    <div className="p-10 bg-black text-white">
      <h1 className="text-3xl">Checkout</h1>

      <button onClick={checkout} className="mt-6 px-6 py-3 bg-white text-black">
        Pay with Stripe
      </button>
    </div>
  );
}

/*****************************************************************
🚀 ENV VARIABLES (.env.local)
*****************************************************************

STRIPE_SECRET_KEY=sk_live_xxx
NEXT_PUBLIC_URL=https://your-domain.com

*****************************************************************
🚀 DEPLOY (VERCEL)
*****************************************************************

1. Push to GitHub
2. Import into Vercel
3. Add env variables
4. Deploy

DONE → LIVE STORE
*/