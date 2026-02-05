# Optional: Realtime (Supabase Realtime)

> **Feature Type:** OPTIONAL
> **Complexity:** Medium
> **Dependencies:** Supabase (Realtime enabled)
> **Agent:** Realtime Agent ⚡
> **Ziel:** Live Updates mit Supabase Realtime Subscriptions

---

## Kontext

**Produkt:** SoundScout Explore
**Entities für Realtime:** Event, Artist, Venue

---

## Wann Realtime nutzen?

- Chat/Messaging Features
- Live Notifications
- Collaborative Editing
- Real-time Dashboards
- Live Activity Feeds

---

## Tasks

### Task R.1: Realtime aktivieren

**In Supabase Dashboard:**

1. Gehe zu Database > Replication
2. Aktiviere Realtime für gewünschte Tabellen

**Oder per SQL:**

```sql
-- Realtime für eine Tabelle aktivieren
ALTER PUBLICATION supabase_realtime ADD TABLE events;

-- Für mehrere Tabellen
ALTER PUBLICATION supabase_realtime ADD TABLE events;
ALTER PUBLICATION supabase_realtime ADD TABLE artists;
ALTER PUBLICATION supabase_realtime ADD TABLE venues;
```

- [ ] Realtime für Tabellen aktiviert

### Task R.2: Realtime Hook

**Datei:** `hooks/useRealtime.ts`

```typescript
"use client";

import { useEffect, useState } from "react";
import { createClient } from "@/lib/supabase/client";
import type { RealtimeChannel, RealtimePostgresChangesPayload } from "@supabase/supabase-js";

type PostgresChange<T> = RealtimePostgresChangesPayload<T>;

interface UseRealtimeOptions<T> {
  table: string;
  filter?: string;
  onInsert?: (payload: PostgresChange<T>) => void;
  onUpdate?: (payload: PostgresChange<T>) => void;
  onDelete?: (payload: PostgresChange<T>) => void;
}

export function useRealtime<T extends Record<string, unknown>>({
  table,
  filter,
  onInsert,
  onUpdate,
  onDelete,
}: UseRealtimeOptions<T>) {
  const [channel, setChannel] = useState<RealtimeChannel | null>(null);

  useEffect(() => {
    const supabase = createClient();

    const channelConfig: Parameters<typeof supabase.channel>[1] = {
      config: {
        postgres_changes: [
          {
            event: "*",
            schema: "public",
            table,
            filter,
          },
        ],
      },
    };

    const newChannel = supabase
      .channel(`realtime-${table}`, channelConfig)
      .on(
        "postgres_changes",
        { event: "INSERT", schema: "public", table, filter },
        (payload) => onInsert?.(payload as PostgresChange<T>)
      )
      .on(
        "postgres_changes",
        { event: "UPDATE", schema: "public", table, filter },
        (payload) => onUpdate?.(payload as PostgresChange<T>)
      )
      .on(
        "postgres_changes",
        { event: "DELETE", schema: "public", table, filter },
        (payload) => onDelete?.(payload as PostgresChange<T>)
      )
      .subscribe();

    setChannel(newChannel);

    return () => {
      supabase.removeChannel(newChannel);
    };
  }, [table, filter]);

  return channel;
}
```

- [ ] Realtime Hook erstellt

### Task R.3: Presence Hook (für Online Status)

**Datei:** `hooks/usePresence.ts`

```typescript
"use client";

import { useEffect, useState } from "react";
import { createClient } from "@/lib/supabase/client";
import type { RealtimeChannel } from "@supabase/supabase-js";

interface PresenceState {
  id: string;
  email?: string;
  online_at: string;
}

export function usePresence(roomId: string, userId: string, userEmail?: string) {
  const [onlineUsers, setOnlineUsers] = useState<PresenceState[]>([]);
  const [channel, setChannel] = useState<RealtimeChannel | null>(null);

  useEffect(() => {
    const supabase = createClient();

    const newChannel = supabase.channel(`presence-${roomId}`);

    newChannel
      .on("presence", { event: "sync" }, () => {
        const state = newChannel.presenceState<PresenceState>();
        const users = Object.values(state).flat();
        setOnlineUsers(users);
      })
      .on("presence", { event: "join" }, ({ newPresences }) => {
        console.log("User joined:", newPresences);
      })
      .on("presence", { event: "leave" }, ({ leftPresences }) => {
        console.log("User left:", leftPresences);
      })
      .subscribe(async (status) => {
        if (status === "SUBSCRIBED") {
          await newChannel.track({
            id: userId,
            email: userEmail,
            online_at: new Date().toISOString(),
          });
        }
      });

    setChannel(newChannel);

    return () => {
      supabase.removeChannel(newChannel);
    };
  }, [roomId, userId, userEmail]);

  return { onlineUsers, channel };
}
```

- [ ] Presence Hook erstellt

### Task R.4: Realtime List Component

**Datei:** `components/realtime/RealtimeList.tsx`

```typescript
"use client";

import { useState, useEffect } from "react";
import { useRealtime } from "@/hooks/useRealtime";

interface RealtimeListProps<T> {
  table: string;
  initialItems: T[];
  renderItem: (item: T) => React.ReactNode;
  filter?: string;
}

export function RealtimeList<T extends { id: string }>({
  table,
  initialItems,
  renderItem,
  filter,
}: RealtimeListProps<T>) {
  const [items, setItems] = useState<T[]>(initialItems);

  useRealtime<T>({
    table,
    filter,
    onInsert: (payload) => {
      const newItem = payload.new as T;
      setItems((prev) => [newItem, ...prev]);
    },
    onUpdate: (payload) => {
      const updatedItem = payload.new as T;
      setItems((prev) =>
        prev.map((item) =>
          item.id === updatedItem.id ? updatedItem : item
        )
      );
    },
    onDelete: (payload) => {
      const deletedItem = payload.old as T;
      setItems((prev) =>
        prev.filter((item) => item.id !== deletedItem.id)
      );
    },
  });

  return (
    <div className="space-y-2">
      {items.map((item) => (
        <div key={item.id}>{renderItem(item)}</div>
      ))}
    </div>
  );
}
```

- [ ] RealtimeList Component erstellt

### Task R.5: Online Users Component

**Datei:** `components/realtime/OnlineUsers.tsx`

```typescript
"use client";

import { usePresence } from "@/hooks/usePresence";

interface OnlineUsersProps {
  roomId: string;
  userId: string;
  userEmail?: string;
}

export function OnlineUsers({ roomId, userId, userEmail }: OnlineUsersProps) {
  const { onlineUsers } = usePresence(roomId, userId, userEmail);

  return (
    <div className="flex items-center gap-2">
      <div className="flex -space-x-2">
        {onlineUsers.slice(0, 5).map((user) => (
          <div
            key={user.id}
            className="w-8 h-8 rounded-full bg-primary flex items-center justify-center text-primary-foreground text-xs font-medium border-2 border-background"
            title={user.email || user.id}
          >
            {(user.email || user.id).charAt(0).toUpperCase()}
          </div>
        ))}
      </div>
      {onlineUsers.length > 5 && (
        <span className="text-sm text-muted-foreground">
          +{onlineUsers.length - 5} more
        </span>
      )}
      <span className="text-sm text-muted-foreground">
        {onlineUsers.length} online
      </span>
    </div>
  );
}
```

- [ ] OnlineUsers Component erstellt

### Task R.6: Realtime Notifications

**Datei:** `components/realtime/RealtimeNotifications.tsx`

```typescript
"use client";

import { useEffect, useState } from "react";
import { useRealtime } from "@/hooks/useRealtime";
import { Bell } from "lucide-react";
import { Button } from "@/components/ui/button";

interface Notification {
  id: string;
  user_id: string;
  message: string;
  read: boolean;
  created_at: string;
}

interface RealtimeNotificationsProps {
  userId: string;
}

export function RealtimeNotifications({ userId }: RealtimeNotificationsProps) {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  useRealtime<Notification>({
    table: "notifications",
    filter: `user_id=eq.${userId}`,
    onInsert: (payload) => {
      const newNotification = payload.new as Notification;
      setNotifications((prev) => [newNotification, ...prev]);
      if (!newNotification.read) {
        setUnreadCount((prev) => prev + 1);
      }
    },
    onUpdate: (payload) => {
      const updated = payload.new as Notification;
      setNotifications((prev) =>
        prev.map((n) => (n.id === updated.id ? updated : n))
      );
      // Recalculate unread count
      setNotifications((prev) => {
        setUnreadCount(prev.filter((n) => !n.read).length);
        return prev;
      });
    },
  });

  return (
    <div className="relative">
      <Button variant="ghost" size="icon">
        <Bell className="h-5 w-5" />
        {unreadCount > 0 && (
          <span className="absolute -top-1 -right-1 h-5 w-5 rounded-full bg-destructive text-destructive-foreground text-xs flex items-center justify-center">
            {unreadCount > 9 ? "9+" : unreadCount}
          </span>
        )}
      </Button>
    </div>
  );
}
```

- [ ] RealtimeNotifications Component erstellt

### Task R.7: Notifications Table (falls benötigt)

```sql
-- Notifications Tabelle
CREATE TABLE IF NOT EXISTS notifications (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  message TEXT NOT NULL,
  read BOOLEAN DEFAULT FALSE,
  data JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read own notifications"
ON notifications FOR SELECT
TO authenticated
USING (user_id = auth.uid());

CREATE POLICY "System can insert notifications"
ON notifications FOR INSERT
TO authenticated
WITH CHECK (true);

CREATE POLICY "Users can update own notifications"
ON notifications FOR UPDATE
TO authenticated
USING (user_id = auth.uid());

-- Realtime aktivieren
ALTER PUBLICATION supabase_realtime ADD TABLE notifications;

-- Index
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
```

- [ ] Notifications Table erstellt (falls benötigt)

---

## Usage Examples

### Live-Updates für Entity Liste

```typescript
// In einer Page
import { RealtimeList } from "@/components/realtime/RealtimeList";

export default async function ItemsPage() {
  const items = await fetchItems(); // Initial fetch

  return (
    <RealtimeList
      table="events"
      initialItems={items}
      renderItem={(item) => <ItemCard item={item} />}
    />
  );
}
```

### Online Users in einem Raum

```typescript
// In einer Page
import { OnlineUsers } from "@/components/realtime/OnlineUsers";

export default function RoomPage({ roomId, user }) {
  return (
    <div>
      <OnlineUsers
        roomId={roomId}
        userId={user.id}
        userEmail={user.email}
      />
      {/* Rest of page */}
    </div>
  );
}
```

---

## Checkpoint

**Realtime ist abgeschlossen wenn:**

- [ ] Realtime für Tabellen aktiviert
- [ ] useRealtime Hook funktioniert
- [ ] Live Updates werden empfangen
- [ ] Presence/Online Status funktioniert (falls benötigt)

**Test:**
1. Öffne App in zwei Browser-Tabs
2. Erstelle Item in Tab 1
3. Item erscheint sofort in Tab 2
4. Aktualisiere Item → Updates live
5. Lösche Item → Verschwindet live

---

**Diese Datei ist optional. Nur umsetzen wenn Realtime Features benötigt werden.**
