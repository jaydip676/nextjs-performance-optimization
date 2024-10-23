# Next.js App Router: Complete Performance Optimization Guide

## A Comprehensive Guide to Achieving Perfect Performance Scores

## Table of Contents

1. [Initial Setup and Configuration](#initial-setup-and-configuration)
2. [Metadata and SEO Optimization](#metadata-and-seo-optimization)
3. [Image Optimization Strategies](#image-optimization-strategies)
4. [JavaScript Optimization](#javascript-optimization)
5. [CSS and Styling Optimization](#css-and-styling-optimization)
6. [Server-Side Optimization](#server-side-optimization)
7. [Route and Component Optimization](#route-and-component-optimization)
8. [Third-Party Script Management](#third-party-script-management)
9. [Performance Monitoring](#performance-monitoring)
10. [Advanced Optimization Techniques](#advanced-optimization-techniques)

## Initial Setup and Configuration

### Next.js Configuration

Create a properly optimized `next.config.js`:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    formats: ["image/avif", "image/webp"],
    remotePatterns: [
      {
        protocol: "https",
        hostname: "**",
      },
    ],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  compress: true,
  poweredByHeader: false,
  reactStrictMode: true,
  compiler: {
    removeConsole: process.env.NODE_ENV === "production",
  },
  experimental: {
    optimizeCss: true,
    turbo: {
      loaders: {
        ".svg": ["@svgr/webpack"],
      },
    },
  },
};

// Enable response compression
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
});

module.exports = withBundleAnalyzer(nextConfig);
```

### Environment Configuration

Create a `.env.production` file:

```env
NODE_ENV=production
NEXT_PUBLIC_API_URL=your_api_url
NEXT_PUBLIC_ANALYTICS_ID=your_analytics_id
```

## Metadata and SEO Optimization

### Root Layout Configuration

```typescript
// app/layout.tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  metadataBase: new URL("https://yourwebsite.com"),
  title: {
    template: "%s | Your Website",
    default: "Your Website",
  },
  description: "Your website description",
  keywords: ["nextjs", "your", "keywords"],
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      "max-video-preview": -1,
      "max-image-preview": "large",
      "max-snippet": -1,
    },
  },
  verification: {
    google: "your-google-verification-code",
  },
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

## Image Optimization Strategies

### Next/Image Implementation

```typescript
// components/OptimizedImage.tsx
import Image from "next/image";
import { useState } from "react";

interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  priority?: boolean;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
}: OptimizedImageProps) {
  const [isLoading, setLoading] = useState(true);

  return (
    <div className="aspect-w-16 aspect-h-9 relative overflow-hidden rounded-lg">
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        quality={90}
        className={`
          duration-700 ease-in-out
          ${
            isLoading
              ? "scale-110 blur-2xl grayscale"
              : "scale-100 blur-0 grayscale-0"
          }
        `}
        onLoadingComplete={() => setLoading(false)}
        sizes="(max-width: 640px) 100vw,
               (max-width: 1024px) 50vw,
               33vw"
      />
    </div>
  );
}
```

### Image Loading Strategy

```typescript
// utils/imageLoader.ts
export const imageLoader = ({ src, width, quality }) => {
  if (src.startsWith("data:") || src.startsWith("blob:")) return src;
  return `https://your-image-cdn.com/${src}?w=${width}&q=${quality || 75}`;
};
```

## JavaScript Optimization

### Bundle Size Optimization

```typescript
// next.config.js additional configuration
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
  openAnalyzer: true,
});

module.exports = withBundleAnalyzer({
  webpack: (config, { dev, isServer }) => {
    // Split chunks optimization
    if (!dev && !isServer) {
      config.optimization.splitChunks = {
        chunks: "all",
        minSize: 20000,
        maxSize: 244000,
        minChunks: 1,
        maxAsyncRequests: 30,
        maxInitialRequests: 30,
        cacheGroups: {
          default: false,
          vendors: false,
          framework: {
            chunks: "all",
            name: "framework",
            test: /(?<!node_modules.*)[\\/]node_modules[\\/](react|react-dom|scheduler|prop-types|use-subscription)[\\/]/,
            priority: 40,
            enforce: true,
          },
          lib: {
            test(module) {
              return (
                module.size() > 160000 &&
                /node_modules[/\\]/.test(module.identifier())
              );
            },
            name(module) {
              const hash = crypto.createHash("sha1");
              hash.update(module.identifier());
              return hash.digest("hex").substring(0, 8);
            },
            priority: 30,
            minChunks: 1,
            reuseExistingChunk: true,
          },
          commons: {
            name: "commons",
            minChunks: 2,
            priority: 20,
          },
          shared: {
            name(module, chunks) {
              return (
                crypto
                  .createHash("sha1")
                  .update(chunks.map((c) => c.name).join("_"))
                  .digest("hex") + "_shared"
              );
            },
            priority: 10,
            minChunks: 2,
            reuseExistingChunk: true,
          },
        },
      };
    }
    return config;
  },
});
```

### Dynamic Imports Strategy

```typescript
// components/DynamicComponentLoader.tsx
import dynamic from "next/dynamic";
import { Suspense } from "react";

const DynamicComponent = dynamic(
  () => import("./HeavyComponent").then((mod) => mod.HeavyComponent),
  {
    loading: () => <LoadingSpinner />,
    ssr: false,
  }
);

export function DynamicComponentLoader() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <DynamicComponent />
    </Suspense>
  );
}
```

## CSS and Styling Optimization

### Tailwind Configuration

```javascript
// tailwind.config.js
const defaultTheme = require("tailwindcss/defaultTheme");

module.exports = {
  content: ["./app/**/*.{js,ts,jsx,tsx}", "./components/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {
      fontFamily: {
        sans: ["Inter var", ...defaultTheme.fontFamily.sans],
      },
    },
  },
  plugins: [
    require("@tailwindcss/forms"),
    require("@tailwindcss/aspect-ratio"),
  ],
};
```

### Critical CSS Extraction

```typescript
// app/head.tsx
export default function Head() {
  return (
    <>
      <style
        dangerouslySetInnerHTML={{
          __html: `
            /* Critical CSS here */
            @font-face {
              font-family: 'Inter var';
              font-weight: 100 900;
              font-display: swap;
              font-style: normal;
              font-named-instance: 'Regular';
              src: url("/fonts/Inter-roman.var.woff2") format("woff2");
            }
          `,
        }}
      />
    </>
  );
}
```

## Server-Side Optimization

### API Route Optimization

```typescript
// app/api/data/route.ts
import { NextResponse } from "next/server";
import { headers } from "next/headers";

export const runtime = "edge";

export async function GET() {
  const headersList = headers();
  const cachedData = await redis.get("cached_data");

  if (cachedData) {
    return NextResponse.json(JSON.parse(cachedData), {
      headers: {
        "Cache-Control": "public, s-maxage=3600, stale-while-revalidate=86400",
        "CDN-Cache-Control":
          "public, s-maxage=3600, stale-while-revalidate=86400",
      },
    });
  }

  const data = await fetchData();
  await redis.set("cached_data", JSON.stringify(data), "EX", 3600);

  return NextResponse.json(data, {
    headers: {
      "Cache-Control": "public, s-maxage=3600, stale-while-revalidate=86400",
      "CDN-Cache-Control":
        "public, s-maxage=3600, stale-while-revalidate=86400",
    },
  });
}
```

### Data Fetching Strategy

```typescript
// utils/fetchWithCache.ts
export async function fetchWithCache(url: string, options: RequestInit = {}) {
  const cacheKey = `data-cache-${url}`;
  const cachedResponse = await caches.match(cacheKey);

  if (cachedResponse) {
    const data = await cachedResponse.json();
    if (data.expires > Date.now()) {
      return data;
    }
  }

  const response = await fetch(url, {
    ...options,
    next: { revalidate: 3600 },
  });
  const data = await response.json();

  const cache = await caches.open("data-cache");
  await cache.put(
    cacheKey,
    new Response(
      JSON.stringify({
        ...data,
        expires: Date.now() + 3600000,
      })
    )
  );

  return data;
}
```

## Route and Component Optimization

### Route Group Organization

```
app/
  (auth)/
    login/
      page.tsx
    register/
      page.tsx
  (dashboard)/
    overview/
      page.tsx
    settings/
      page.tsx
  (marketing)/
    blog/
      page.tsx
    about/
      page.tsx
```

### Parallel Routes Implementation

```typescript
// app/layout.tsx
export default function Layout({
  children,
  modal,
  auth,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
  auth: React.ReactNode;
}) {
  return (
    <html>
      <body>
        {children}
        {modal}
        {auth}
      </body>
    </html>
  );
}
```

## Third-Party Script Management

### Script Loading Optimization

```typescript
// app/layout.tsx
import Script from "next/script";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}

        {/* Analytics */}
        <Script
          strategy="afterInteractive"
          src={`https://www.googletagmanager.com/gtag/js?id=${process.env.NEXT_PUBLIC_GA_ID}`}
        />
        <Script id="google-analytics" strategy="afterInteractive">
          {`
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', '${process.env.NEXT_PUBLIC_GA_ID}');
          `}
        </Script>

        {/* Other third-party scripts */}
        <Script
          src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"
          strategy="lazyOnload"
        />
      </body>
    </html>
  );
}
```

## Performance Monitoring

### Web Vitals Tracking

```typescript
// app/layout.tsx
import { useReportWebVitals } from "next/web-vitals";

export function WebVitals() {
  useReportWebVitals((metric) => {
    const { id, name, label, value } = metric;

    // Analytics
    gtag("event", name, {
      event_category:
        label === "web-vital" ? "Web Vitals" : "Next.js custom metric",
      value: Math.round(name === "CLS" ? value * 1000 : value),
      event_label: id,
      non_interaction: true,
    });
  });

  return null;
}
```

### Performance Budget Configuration

```javascript
// next.config.js
module.exports = {
  experimental: {
    performanceBudget: {
      // Performance budget thresholds
      maxDuration: 300, // Time in milliseconds
      maxSize: {
        javascript: 200000, // Size in bytes
        css: 50000,
        images: 300000,
        fonts: 100000,
        videos: 500000,
      },
    },
  },
};
```

## Advanced Next.js Performance Optimization Techniques

### Table of Contents

1. [Service Worker Implementation](#service-worker-implementation)
2. [Memory Management](#memory-management)
3. [Code Splitting Strategies](#code-splitting-strategies)
4. [Resource Prefetching](#resource-prefetching)
5. [Runtime Performance](#runtime-performance)
6. [State Management Optimization](#state-management-optimization)
7. [Network Optimization](#network-optimization)
8. [Font Optimization](#font-optimization)
9. [Critical Rendering Path](#critical-rendering-path)
10. [Error Boundary Implementation](#error-boundary-implementation)

### Service Worker Implementation

#### Custom Service Worker

```typescript
// public/sw.js
const CACHE_VERSION = "v1.0.0";
const CACHE_NAME = `app-cache-${CACHE_VERSION}`;

const STATIC_CACHE_URLS = [
  "/",
  "/offline",
  "/favicon.ico",
  "/manifest.json",
  "/static/fonts/inter-var.woff2",
];

// Install Service Worker
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(STATIC_CACHE_URLS);
    })
  );
  self.skipWaiting();
});

// Activate and Clean Old Caches
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter(
            (name) => name.startsWith("app-cache-") && name !== CACHE_NAME
          )
          .map((name) => caches.delete(name))
      );
    })
  );
  self.clients.claim();
});

// Advanced Fetch Strategy
self.addEventListener("fetch", (event) => {
  event.respondWith(
    (async () => {
      // API calls strategy
      if (event.request.url.includes("/api/")) {
        try {
          const response = await fetch(event.request);
          if (response.ok) {
            const cache = await caches.open(CACHE_NAME);
            cache.put(event.request, response.clone());
            return response;
          }
          return caches.match(event.request);
        } catch (error) {
          return caches.match(event.request);
        }
      }

      // Static assets strategy
      if (
        event.request.destination === "image" ||
        event.request.destination === "style" ||
        event.request.destination === "script"
      ) {
        return caches.match(event.request).then((response) => {
          return (
            response ||
            fetch(event.request).then((response) => {
              const responseClone = response.clone();
              caches.open(CACHE_NAME).then((cache) => {
                cache.put(event.request, responseClone);
              });
              return response;
            })
          );
        });
      }

      // Network-first strategy for HTML
      if (event.request.mode === "navigate") {
        try {
          const response = await fetch(event.request);
          const cache = await caches.open(CACHE_NAME);
          cache.put(event.request, response.clone());
          return response;
        } catch (error) {
          const cached = await caches.match(event.request);
          return cached || caches.match("/offline");
        }
      }

      return fetch(event.request);
    })()
  );
});
```

#### Service Worker Registration

```typescript
// app/ServiceWorkerRegistration.tsx
"use client";

import { useEffect } from "react";

export function ServiceWorkerRegistration() {
  useEffect(() => {
    if ("serviceWorker" in navigator && process.env.NODE_ENV === "production") {
      window.addEventListener("load", () => {
        navigator.serviceWorker
          .register("/sw.js")
          .then((registration) => {
            console.log("SW registered:", registration);

            // Handle updates
            registration.addEventListener("updatefound", () => {
              const newWorker = registration.installing;
              newWorker?.addEventListener("statechange", () => {
                if (
                  newWorker.state === "installed" &&
                  navigator.serviceWorker.controller
                ) {
                  // New content is available
                  if (confirm("New version available! Reload to update?")) {
                    window.location.reload();
                  }
                }
              });
            });
          })
          .catch((error) => {
            console.error("SW registration failed:", error);
          });
      });
    }
  }, []);

  return null;
}
```

### Memory Management

#### Memory Leak Prevention

```typescript
// hooks/useMemoryOptimization.ts
import { useEffect, useRef, useCallback } from "react";

interface CleanupFunction {
  (): void;
  description?: string;
}

export function useMemoryOptimization() {
  const cleanupFunctions = useRef<CleanupFunction[]>([]);
  const intervals = useRef<number[]>([]);
  const timeouts = useRef<number[]>([]);

  // Register cleanup function
  const registerCleanup = useCallback((cleanup: CleanupFunction) => {
    cleanupFunctions.current.push(cleanup);
  }, []);

  // Safe interval
  const setOptimizedInterval = useCallback(
    (callback: () => void, delay: number) => {
      const id = window.setInterval(callback, delay);
      intervals.current.push(id);
      return id;
    },
    []
  );

  // Safe timeout
  const setOptimizedTimeout = useCallback(
    (callback: () => void, delay: number) => {
      const id = window.setTimeout(callback, delay);
      timeouts.current.push(id);
      return id;
    },
    []
  );

  // Clear specific interval
  const clearOptimizedInterval = useCallback((id: number) => {
    intervals.current = intervals.current.filter((intervalId) => {
      if (intervalId === id) {
        clearInterval(id);
        return false;
      }
      return true;
    });
  }, []);

  // Clear specific timeout
  const clearOptimizedTimeout = useCallback((id: number) => {
    timeouts.current = timeouts.current.filter((timeoutId) => {
      if (timeoutId === id) {
        clearTimeout(id);
        return false;
      }
      return true;
    });
  }, []);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      // Clear all intervals
      intervals.current.forEach((id) => clearInterval(id));
      intervals.current = [];

      // Clear all timeouts
      timeouts.current.forEach((id) => clearTimeout(id));
      timeouts.current = [];

      // Run registered cleanup functions
      cleanupFunctions.current.forEach((cleanup) => {
        try {
          cleanup();
        } catch (error) {
          console.error(
            `Error in cleanup function${
              cleanup.description ? ` (${cleanup.description})` : ""
            }:`,
            error
          );
        }
      });
      cleanupFunctions.current = [];
    };
  }, []);

  return {
    registerCleanup,
    setOptimizedInterval,
    setOptimizedTimeout,
    clearOptimizedInterval,
    clearOptimizedTimeout,
  };
}
```

#### Event Listener Management

```typescript
// hooks/useEventListener.ts
import { useEffect, useRef } from "react";

type EventListener = (event: Event) => void;

export function useEventListener(
  eventName: string,
  handler: EventListener,
  element: Window | Element | null = window,
  options?: AddEventListenerOptions
) {
  const savedHandler = useRef<EventListener>();
  const isSupported = element && element.addEventListener;

  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);

  useEffect(() => {
    if (!isSupported) return;

    const eventListener: EventListener = (event) => {
      savedHandler.current?.(event);
    };

    element.addEventListener(eventName, eventListener, options);

    return () => {
      element.removeEventListener(eventName, eventListener, options);
    };
  }, [eventName, element, isSupported, options]);
}
```

### Code Splitting Strategies

#### Component-Level Code Splitting

```typescript
// components/DynamicImports.tsx
import dynamic from "next/dynamic";

// Heavy components with custom loading
export const HeavyChart = dynamic(
  () => import("./Chart").then((mod) => mod.Chart),
  {
    loading: () => <ChartSkeleton />,
    ssr: false,
  }
);

// Dynamic import with preload
export const Modal = dynamic(() => {
  return import("./Modal")
    .then((mod) => {
      // Preload related components
      const CompA = import("./CompA");
      const CompB = import("./CompB");
      return Promise.all([mod.default, CompA, CompB]);
    })
    .then(([ModalComponent]) => ModalComponent);
});

// Route-based code splitting
export const AdminDashboard = dynamic(() => import("./AdminDashboard"), {
  loading: () => <AdminSkeleton />,
  ssr: true,
});
```

#### Module Splitting Configuration

```typescript
// next.config.js
module.exports = {
  webpack: (config, { dev, isServer }) => {
    // Optimize code splitting
    if (!dev && !isServer) {
      config.optimization.splitChunks = {
        chunks: "all",
        minSize: 20000,
        maxSize: 244000,
        minChunks: 1,
        maxAsyncRequests: 30,
        maxInitialRequests: 30,
        enforceSizeThreshold: 50000,
        cacheGroups: {
          default: false,
          vendors: false,
          // Framework chunk
          framework: {
            chunks: "all",
            name: "framework",
            test: /(?<!node_modules.*)[\\/]node_modules[\\/](react|react-dom|scheduler|prop-types|use-subscription)[\\/]/,
            priority: 40,
            enforce: true,
          },
          // Library chunk
          lib: {
            test(module) {
              return (
                module.size() > 160000 &&
                /node_modules[/\\]/.test(module.identifier())
              );
            },
            name(module) {
              const hash = crypto.createHash("sha1");
              hash.update(module.identifier());
              return hash.digest("hex").substring(0, 8);
            },
            priority: 30,
            minChunks: 1,
            reuseExistingChunk: true,
          },
          // Commons chunk
          commons: {
            name: "commons",
            minChunks: 2,
            priority: 20,
          },
          // Shared chunk
          shared: {
            name(module, chunks) {
              return (
                crypto
                  .createHash("sha1")
                  .update(chunks.reduce((acc, chunk) => acc + chunk.name, ""))
                  .digest("hex") + "_shared"
              );
            },
            priority: 10,
            minChunks: 2,
            reuseExistingChunk: true,
          },
        },
      };

      // Enable module concatenation
      config.optimization.concatenateModules = true;

      // Enable deterministic chunk ids
      config.optimization.chunkIds = "deterministic";
    }

    return config;
  },
};
```

### Resource Prefetching

#### Link Prefetching Strategy

```typescript
// components/OptimizedLink.tsx
import { useCallback, useEffect, useRef } from "react";
import Link from "next/link";
import { useRouter } from "next/router";

interface OptimizedLinkProps {
  href: string;
  children: React.ReactNode;
  prefetch?: boolean;
  prefetchTimeout?: number;
}

export function OptimizedLink({
  href,
  children,
  prefetch = true,
  prefetchTimeout = 2000,
}: OptimizedLinkProps) {
  const router = useRouter();
  const timeoutRef = useRef<NodeJS.Timeout>();
  const isMounted = useRef(true);

  const prefetchPage = useCallback(() => {
    if (prefetch) {
      timeoutRef.current = setTimeout(() => {
        if (isMounted.current) {
          router.prefetch(href);
        }
      }, prefetchTimeout);
    }
  }, [href, prefetch, prefetchTimeout, router]);

  useEffect(() => {
    return () => {
      isMounted.current = false;
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return (
    <Link href={href} onMouseEnter={prefetchPage} onTouchStart={prefetchPage}>
      {children}
    </Link>
  );
}
```

#### Resource Hints Implementation

```typescript
// app/head.tsx
export default function Head() {
  return (
    <>
      {/* DNS Prefetch */}
      <link rel="dns-prefetch" href="https://fonts.googleapis.com" />

      {/* Preconnect */}
      <link
        rel="preconnect"
        href="https://fonts.gstatic.com"
        crossOrigin="anonymous"
      />

      {/* Preload Critical Resources */}
      <link
        rel="preload"
        href="/fonts/inter-var.woff2"
        as="font"
        type="font/woff2"
        crossOrigin="anonymous"
      />

      {/* Prefetch Important Pages */}
      <link rel="prefetch" href="/api/critical-data" />

      {/* Prerender Next Likely Page */}
      <link rel="prerender" href="/dashboard" />
    </>
  );
}
```

### Runtime Performance

#### React Memo Implementation

```typescript
// components/OptimizedList.tsx
import { memo, useMemo, useState, useCallback } from "react";

interface ListItemProps {
  item: {
    id: string;
    title: string;
  };
  onSelect: (id: string) => void;
}

const ListItem = memo(
  function ListItem({ item, onSelect }: ListItemProps) {
    return (
      <div
        onClick={() => onSelect(item.id)}
        className="p-4 hover:bg-gray-50 transition-colors"
      >
        {item.title}
      </div>
    );
  },
  (prevProps, nextProps) => {
    return (
      prevProps.item.id === nextProps.item.id &&
      prevProps.item.title === nextProps.item.title
    );
  }
);

export function OptimizedList({ items }) {
  const [selectedId, setSelectedId] = useState<string | null>(null);

  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => a.title.localeCompare(b.title));
  }, [items]);

  const handleSelect = useCallback((id: string) => {
    setSelectedId(id);
  }, []);

  return (
    <div>
      {sortedItems.map((item) => (
        <ListItem key={item.id} item={item} onSelect={handleSelect} />
      ))}
    </div>
  );
}
```

#### Virtual List Implementation

```typescript
// components/VirtualList.tsx (continued)
interface VirtualListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  itemHeight: number;
  overscan?: number;
}

export function VirtualList<T>({
  items,
  renderItem,
  itemHeight,
  overscan = 3,
}: VirtualListProps<T>) {
  const parentRef = useRef<HTMLDivElement>(null);
  const [parentHeight, setParentHeight] = useState(0);

  useEffect(() => {
    if (parentRef.current) {
      const resizeObserver = new ResizeObserver((entries) => {
        setParentHeight(entries[0].contentRect.height);
      });
      resizeObserver.observe(parentRef.current);
      return () => resizeObserver.disconnect();
    }
  }, []);

  const rowVirtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => itemHeight,
    overscan,
  });

  return (
    <div
      ref={parentRef}
      className="h-full overflow-auto"
      style={{ contain: "strict" }}
    >
      <div
        style={{
          height: `${rowVirtualizer.getTotalSize()}px`,
          width: "100%",
          position: "relative",
        }}
      >
        {rowVirtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.index}
            style={{
              position: "absolute",
              top: 0,
              left: 0,
              width: "100%",
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {renderItem(items[virtualItem.index], virtualItem.index)}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### State Management Optimization

#### Context Optimization

```typescript
// context/OptimizedContext.tsx
import { createContext, useContext, useCallback, useMemo, useRef } from "react";
import { produce } from "immer";

interface State {
  data: Record<string, any>;
  cache: Map<string, any>;
}

interface ContextValue {
  state: State;
  updateData: (key: string, value: any) => void;
  clearCache: () => void;
}

const StateContext = createContext<ContextValue | null>(null);

export function StateProvider({ children }: { children: React.ReactNode }) {
  const stateRef = useRef<State>({
    data: {},
    cache: new Map(),
  });

  const updateData = useCallback((key: string, value: any) => {
    stateRef.current = produce(stateRef.current, (draft) => {
      draft.data[key] = value;
    });
  }, []);

  const clearCache = useCallback(() => {
    stateRef.current.cache.clear();
  }, []);

  const value = useMemo(
    () => ({
      state: stateRef.current,
      updateData,
      clearCache,
    }),
    [updateData, clearCache]
  );

  return (
    <StateContext.Provider value={value}>{children}</StateContext.Provider>
  );
}

export function useOptimizedState() {
  const context = useContext(StateContext);
  if (!context) {
    throw new Error("useOptimizedState must be used within StateProvider");
  }
  return context;
}
```

### Network Optimization

#### Request Batching

```typescript
// utils/requestBatcher.ts
interface BatchRequest {
  endpoint: string;
  method: string;
  body?: any;
  resolve: (value: any) => void;
  reject: (error: any) => void;
}

class RequestBatcher {
  private batchTimeout: number;
  private batchSize: number;
  private pendingRequests: BatchRequest[] = [];
  private timeoutId: NodeJS.Timeout | null = null;

  constructor(batchTimeout = 50, batchSize = 10) {
    this.batchTimeout = batchTimeout;
    this.batchSize = batchSize;
  }

  async add(endpoint: string, method = "GET", body?: any): Promise<any> {
    return new Promise((resolve, reject) => {
      this.pendingRequests.push({
        endpoint,
        method,
        body,
        resolve,
        reject,
      });

      if (this.pendingRequests.length >= this.batchSize) {
        this.flush();
      } else if (!this.timeoutId) {
        this.timeoutId = setTimeout(() => this.flush(), this.batchTimeout);
      }
    });
  }

  private async flush() {
    if (this.timeoutId) {
      clearTimeout(this.timeoutId);
      this.timeoutId = null;
    }

    const requests = [...this.pendingRequests];
    this.pendingRequests = [];

    try {
      const response = await fetch("/api/batch", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(
          requests.map((req) => ({
            endpoint: req.endpoint,
            method: req.method,
            body: req.body,
          }))
        ),
      });

      const results = await response.json();

      requests.forEach((req, index) => {
        if (results[index].error) {
          req.reject(results[index].error);
        } else {
          req.resolve(results[index].data);
        }
      });
    } catch (error) {
      requests.forEach((req) => req.reject(error));
    }
  }
}

export const requestBatcher = new RequestBatcher();
```

#### API Route Handler for Batched Requests

```typescript
// app/api/batch/route.ts
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const requests = await req.json();

  const results = await Promise.all(
    requests.map(async (request: any) => {
      try {
        const response = await fetch(
          `${process.env.API_BASE_URL}${request.endpoint}`,
          {
            method: request.method,
            headers: {
              "Content-Type": "application/json",
            },
            body: request.body ? JSON.stringify(request.body) : undefined,
          }
        );

        const data = await response.json();
        return { data };
      } catch (error) {
        return { error: error.message };
      }
    })
  );

  return NextResponse.json(results);
}
```

### Font Optimization

#### Font Loading Strategy

```typescript
// app/FontOptimization.tsx
export function FontOptimization() {
  return (
    <>
      <link
        rel="preload"
        href="/fonts/inter-var.woff2"
        as="font"
        type="font/woff2"
        crossOrigin="anonymous"
      />
      <style jsx global>{`
        @font-face {
          font-family: "Inter var";
          font-weight: 100 900;
          font-display: optional;
          font-style: normal;
          font-named-instance: "Regular";
          src: url("/fonts/inter-var.woff2") format("woff2");
        }

        @font-face {
          font-family: "Inter var";
          font-weight: 100 900;
          font-display: optional;
          font-style: italic;
          font-named-instance: "Italic";
          src: url("/fonts/inter-var-italic.woff2") format("woff2");
        }
      `}</style>
    </>
  );
}
```

### Critical Rendering Path

#### Critical CSS Extraction

```typescript
// scripts/extractCriticalCSS.js
const critical = require("critical");

async function extractCriticalCSS() {
  try {
    const result = await critical.generate({
      base: "out/",
      src: "index.html",
      target: {
        css: "styles/critical.css",
        html: "index-critical.html",
      },
      width: 1300,
      height: 900,
      inline: true,
      extract: true,
      penthouse: {
        blockJSRequests: false,
      },
    });

    console.log("Critical CSS extracted successfully");
  } catch (error) {
    console.error("Error extracting critical CSS:", error);
  }
}

extractCriticalCSS();
```

#### Implementation in Layout

```typescript
// app/layout.tsx
import { getCriticalCss } from "@/utils/getCriticalCss";

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const criticalCss = await getCriticalCss();

  return (
    <html lang="en">
      <head>
        <style
          dangerouslySetInnerHTML={{ __html: criticalCss }}
          data-critical="true"
        />
      </head>
      <body>
        {children}
        <link
          rel="stylesheet"
          href="/styles/main.css"
          media="print"
          onLoad="this.media='all'"
        />
      </body>
    </html>
  );
}
```

### Error Boundary Implementation

#### Custom Error Boundary

```typescript
// components/ErrorBoundary.tsx
"use client";

import { Component, ErrorInfo, ReactNode } from "react";

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false,
    error: null,
  };

  public static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error("ErrorBoundary caught an error:", error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  public render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div className="error-boundary">
            <h2>Something went wrong.</h2>
            <details style={{ whiteSpace: "pre-wrap" }}>
              {this.state.error?.toString()}
            </details>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

// Usage with monitoring
export function withErrorBoundary<P extends object>(
  WrappedComponent: React.ComponentType<P>,
  options?: {
    fallback?: ReactNode;
    onError?: (error: Error, errorInfo: ErrorInfo) => void;
  }
) {
  return function WithErrorBoundary(props: P) {
    return (
      <ErrorBoundary
        fallback={options?.fallback}
        onError={(error, errorInfo) => {
          // Log to monitoring service
          console.error("Error:", error);
          console.error("Error Info:", errorInfo);
          options?.onError?.(error, errorInfo);
        }}
      >
        <WrappedComponent {...props} />
      </ErrorBoundary>
    );
  };
}
```
