# Frontend Design - Anchor Air Management System

A modern web application for managing Anchor Air's operations using Next.js, shadcn/ui, Tailwind CSS, Drizzle ORM, React Query, and React Table.

## Tech Stack

- **Framework**: Next.js 14+ (App Router)
- **UI Components**: shadcn/ui
- **Styling**: Tailwind CSS
- **ORM**: Drizzle ORM (PostgreSQL)
- **Data Fetching**: TanStack React Query (formerly React Query)
- **Tables**: TanStack React Table
- **Type Safety**: TypeScript

## Application Architecture

### Project Structure

```
anchor-air/
├── app/
│   ├── (dashboard)/
│   │   ├── flights/
│   │   │   ├── page.tsx
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx
│   │   │   └── new/
│   │   │       └── page.tsx
│   │   ├── passengers/
│   │   │   ├── page.tsx
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx
│   │   │   └── bookings/
│   │   │       └── page.tsx
│   │   ├── crew/
│   │   │   ├── page.tsx
│   │   │   ├── pilots/
│   │   │   │   └── page.tsx
│   │   │   ├── attendants/
│   │   │   │   └── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   ├── airports/
│   │   │   ├── page.tsx
│   │   │   └── [code]/
│   │   │       └── page.tsx
│   │   ├── aircraft/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   ├── bookings/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   └── layout.tsx
│   ├── api/
│   │   ├── flights/
│   │   │   ├── route.ts
│   │   │   └── [id]/
│   │   │       └── route.ts
│   │   ├── passengers/
│   │   │   └── route.ts
│   │   ├── crew/
│   │   │   └── route.ts
│   │   ├── bookings/
│   │   │   └── route.ts
│   │   └── airports/
│   │       └── route.ts
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/                    # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   ├── form.tsx
│   │   ├── input.tsx
│   │   ├── select.tsx
│   │   ├── table.tsx
│   │   ├── tabs.tsx
│   │   └── ...
│   ├── layout/
│   │   ├── nav-bar.tsx
│   │   ├── side-bar.tsx
│   │   └── header.tsx
│   ├── flights/
│   │   ├── flight-table.tsx
│   │   ├── flight-form.tsx
│   │   ├── flight-details.tsx
│   │   ├── seat-map.tsx
│   │   └── leg-instance-form.tsx
│   ├── passengers/
│   │   ├── passenger-table.tsx
│   │   ├── passenger-form.tsx
│   │   └── booking-history.tsx
│   ├── crew/
│   │   ├── crew-table.tsx
│   │   ├── crew-assignment-form.tsx
│   │   ├── pilot-certifications.tsx
│   │   └── schedule-calendar.tsx
│   ├── bookings/
│   │   ├── booking-form.tsx
│   │   ├── seat-selector.tsx
│   │   ├── fare-calculator.tsx
│   │   └── booking-confirmation.tsx
│   └── airports/
│       ├── airport-card.tsx
│       ├── gate-availability.tsx
│       └── flight-board.tsx
├── lib/
│   ├── db/
│   │   ├── index.ts
│   │   ├── schema.ts
│   │   └── migrations/
│   ├── api/
│   │   ├── flights.ts
│   │   ├── passengers.ts
│   │   ├── crew.ts
│   │   ├── bookings.ts
│   │   └── airports.ts
│   ├── hooks/
│   │   ├── use-flights.ts
│   │   ├── use-passengers.ts
│   │   ├── use-crew.ts
│   │   └── use-bookings.ts
│   ├── utils.ts
│   └── validations/
│       ├── flight-schema.ts
│       ├── passenger-schema.ts
│       └── booking-schema.ts
├── types/
│   ├── database.ts
│   ├── api.ts
│   └── components.ts
└── drizzle.config.ts
```

## Database Layer (Drizzle ORM)

### Schema Definition (`lib/db/schema.ts`)

```typescript
import { pgTable, varchar, integer, timestamp, date, foreignKey, primaryKey } from 'drizzle-orm/pg-core';

// Airplane Types
export const airplaneTypes = pgTable('airplane_type', {
  typeId: integer('type_id').primaryKey(),
  numSeats: integer('num_seats').notNull(),
  manufacturer: varchar('manufacturer', { length: 255 }).notNull(),
  modelType: varchar('model_type', { length: 255 }).notNull(),
});

// Employees
export const employees = pgTable('employees', {
  empId: varchar('emp_id', { length: 255 }).primaryKey(),
  fname: varchar('fname', { length: 255 }).notNull(),
  lname: varchar('lname', { length: 255 }).notNull(),
  phone: varchar('phone', { length: 255 }),
  email: varchar('email', { length: 255 }),
  address: varchar('address', { length: 255 }),
  empType: varchar('emp_type', { length: 255 }),
});

// Flight Crew
export const flightCrew = pgTable('flight_crew', {
  empId: varchar('emp_id', { length: 255 }).primaryKey().references(() => employees.empId),
  salary: integer('salary').notNull(),
});

// Airports
export const airports = pgTable('airports', {
  airportId: varchar('airport_id', { length: 255 }).primaryKey(),
  airportName: varchar('airport_name', { length: 255 }).notNull(),
  city: varchar('city', { length: 255 }).notNull(),
  crewId: varchar('crew_id', { length: 255 }).notNull().references(() => flightCrew.empId),
});

// Flights
export const flights = pgTable('flights', {
  flightId: varchar('flight_id', { length: 255 }).primaryKey(),
  flightNumber: integer('flight_number').notNull(),
  airlineCode: varchar('airline_code', { length: 255 }).notNull(),
  operationDays: varchar('operation_days', { length: 255 }).notNull(),
  planeTypeId: integer('plane_type_id').notNull().references(() => airplaneTypes.typeId),
});

// Flight Instances
export const flightInstances = pgTable('flight_instance', {
  flightId: varchar('flight_id', { length: 255 }).notNull().references(() => flights.flightId),
  flightDate: date('flight_date').notNull(),
  startLocation: varchar('start_location', { length: 255 }).notNull(),
  destinationLocation: varchar('destination_location', { length: 255 }).notNull(),
  departureTime: timestamp('departure_time').notNull(),
  arrivalTime: timestamp('arrival_time').notNull(),
}, (table) => ({
  pk: primaryKey({ columns: [table.flightId, table.flightDate] }),
}));

// Passengers
export const passengers = pgTable('passengers', {
  passengerId: varchar('passenger_id', { length: 255 }).primaryKey(),
  fname: varchar('fname', { length: 255 }).notNull(),
  lname: varchar('lname', { length: 255 }).notNull(),
  phone: varchar('phone', { length: 255 }),
  email: varchar('email', { length: 255 }),
  address: varchar('address', { length: 255 }),
});

// Seats
export const seats = pgTable('seats', {
  seatNumber: varchar('seat_number', { length: 255 }).primaryKey(),
  flightId: varchar('flight_id', { length: 255 }).notNull(),
  flightDate: date('flight_date').notNull(),
  paymentId: varchar('payment_id', { length: 255 }).notNull(),
  passengerId: varchar('passenger_id', { length: 255 }).references(() => passengers.passengerId),
}, (table) => ({
  flightRef: foreignKey({
    columns: [table.flightId, table.flightDate],
    foreignColumns: [flightInstances.flightId, flightInstances.flightDate],
  }),
}));

// Fare Info
export const fareInfo = pgTable('fare_info', {
  code: varchar('code', { length: 255 }).primaryKey(),
  restrictions: varchar('restrictions', { length: 255 }),
  amountRule: varchar('amount_rule', { length: 255 }),
});

// ... Additional tables following the same pattern
```

### Database Connection (`lib/db/index.ts`)

```typescript
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

const connectionString = process.env.DATABASE_URL!;
const client = postgres(connectionString);

export const db = drizzle(client, { schema });
```

## API Layer (Server Actions & Route Handlers)

### Flight API (`lib/api/flights.ts`)

```typescript
import { db } from '@/lib/db';
import { flights, flightInstances, airplaneTypes } from '@/lib/db/schema';
import { eq, and, gte, lte, desc } from 'drizzle-orm';

export async function getFlights(filters?: {
  date?: Date;
  origin?: string;
  destination?: string;
}) {
  let query = db
    .select({
      flight: flights,
      instance: flightInstances,
      aircraft: airplaneTypes,
    })
    .from(flightInstances)
    .innerJoin(flights, eq(flightInstances.flightId, flights.flightId))
    .innerJoin(airplaneTypes, eq(flights.planeTypeId, airplaneTypes.typeId));

  if (filters?.date) {
    query = query.where(eq(flightInstances.flightDate, filters.date));
  }

  if (filters?.origin) {
    query = query.where(eq(flightInstances.startLocation, filters.origin));
  }

  if (filters?.destination) {
    query = query.where(eq(flightInstances.destinationLocation, filters.destination));
  }

  return await query.orderBy(desc(flightInstances.departureTime));
}

export async function getFlightById(flightId: string, flightDate: Date) {
  const result = await db
    .select()
    .from(flightInstances)
    .where(
      and(
        eq(flightInstances.flightId, flightId),
        eq(flightInstances.flightDate, flightDate)
      )
    )
    .limit(1);

  return result[0];
}

export async function createFlight(data: typeof flights.$inferInsert) {
  const result = await db.insert(flights).values(data).returning();
  return result[0];
}

// Additional CRUD operations...
```

## React Query Hooks

### Flight Hooks (`lib/hooks/use-flights.ts`)

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { getFlights, getFlightById, createFlight } from '@/lib/api/flights';

export function useFlights(filters?: Parameters<typeof getFlights>[0]) {
  return useQuery({
    queryKey: ['flights', filters],
    queryFn: () => getFlights(filters),
  });
}

export function useFlight(flightId: string, flightDate: Date) {
  return useQuery({
    queryKey: ['flight', flightId, flightDate],
    queryFn: () => getFlightById(flightId, flightDate),
    enabled: !!flightId && !!flightDate,
  });
}

export function useCreateFlight() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createFlight,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['flights'] });
    },
  });
}

// Additional mutations for update, delete...
```

## UI Components

### Flight Table with React Table (`components/flights/flight-table.tsx`)

```typescript
'use client';

import { useMemo } from 'react';
import {
  useReactTable,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  flexRender,
  type ColumnDef,
} from '@tanstack/react-table';
import { useFlights } from '@/lib/hooks/use-flights';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';

type Flight = {
  flightId: string;
  flightNumber: number;
  departureTime: Date;
  arrivalTime: Date;
  startLocation: string;
  destinationLocation: string;
  aircraft: {
    manufacturer: string;
    modelType: string;
  };
};

export function FlightTable() {
  const { data: flights, isLoading } = useFlights();

  const columns = useMemo<ColumnDef<Flight>[]>(
    () => [
      {
        accessorKey: 'flightNumber',
        header: 'Flight #',
      },
      {
        accessorKey: 'startLocation',
        header: 'Origin',
      },
      {
        accessorKey: 'destinationLocation',
        header: 'Destination',
      },
      {
        accessorKey: 'departureTime',
        header: 'Departure',
        cell: ({ getValue }) => {
          const date = getValue() as Date;
          return date.toLocaleString();
        },
      },
      {
        accessorKey: 'arrivalTime',
        header: 'Arrival',
        cell: ({ getValue }) => {
          const date = getValue() as Date;
          return date.toLocaleString();
        },
      },
      {
        accessorKey: 'aircraft',
        header: 'Aircraft',
        cell: ({ getValue }) => {
          const aircraft = getValue() as Flight['aircraft'];
          return `${aircraft.manufacturer} ${aircraft.modelType}`;
        },
      },
      {
        id: 'actions',
        cell: ({ row }) => (
          <Button variant="ghost" size="sm">
            View Details
          </Button>
        ),
      },
    ],
    []
  );

  const table = useReactTable({
    data: flights ?? [],
    columns,
    getCoreRowModel: getCoreRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      <Table>
        <TableHeader>
          {table.getHeaderGroups().map((headerGroup) => (
            <TableRow key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <TableHead key={header.id}>
                  {flexRender(
                    header.column.columnDef.header,
                    header.getContext()
                  )}
                </TableHead>
              ))}
            </TableRow>
          ))}
        </TableHeader>
        <TableBody>
          {table.getRowModel().rows.map((row) => (
            <TableRow key={row.id}>
              {row.getVisibleCells().map((cell) => (
                <TableCell key={cell.id}>
                  {flexRender(cell.column.columnDef.cell, cell.getContext())}
                </TableCell>
              ))}
            </TableRow>
          ))}
        </TableBody>
      </Table>
      <div className="flex items-center justify-end space-x-2 py-4">
        <Button
          variant="outline"
          size="sm"
          onClick={() => table.previousPage()}
          disabled={!table.getCanPreviousPage()}
        >
          Previous
        </Button>
        <Button
          variant="outline"
          size="sm"
          onClick={() => table.nextPage()}
          disabled={!table.getCanNextPage()}
        >
          Next
        </Button>
      </div>
    </div>
  );
}
```

### Booking Form with shadcn (`components/bookings/booking-form.tsx`)

```typescript
'use client';

import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import * as z from 'zod';
import { Button } from '@/components/ui/button';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { useCreateBooking } from '@/lib/hooks/use-bookings';

const bookingSchema = z.object({
  passengerId: z.string().min(1, 'Passenger is required'),
  flightId: z.string().min(1, 'Flight is required'),
  flightDate: z.date(),
  seatNumber: z.string().min(1, 'Seat selection is required'),
  fareCode: z.string().min(1, 'Fare class is required'),
});

export function BookingForm() {
  const createBooking = useCreateBooking();

  const form = useForm<z.infer<typeof bookingSchema>>({
    resolver: zodResolver(bookingSchema),
  });

  async function onSubmit(values: z.infer<typeof bookingSchema>) {
    await createBooking.mutateAsync(values);
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <FormField
          control={form.control}
          name="passengerId"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Passenger</FormLabel>
              <FormControl>
                <Input placeholder="Select passenger..." {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="fareCode"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Fare Class</FormLabel>
              <Select onValueChange={field.onChange} defaultValue={field.value}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Select fare class" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="first">First Class</SelectItem>
                  <SelectItem value="business">Business Class</SelectItem>
                  <SelectItem value="coach">Coach Class</SelectItem>
                  <SelectItem value="economy">Economy Class</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={createBooking.isPending}>
          {createBooking.isPending ? 'Creating...' : 'Create Booking'}
        </Button>
      </form>
    </Form>
  );
}
```

## Key Features by Module

### 1. Flight Management
- **Flight Schedule Board**: Real-time view of all scheduled flights
- **Flight Details**: Complete flight information with leg breakdowns
- **Seat Map**: Visual seat selection with availability status
- **Multi-Leg Routing**: Support for flights with layovers

### 2. Passenger Management
- **Passenger Directory**: Searchable, sortable table of all passengers
- **Booking History**: View all bookings for a passenger
- **Profile Management**: Edit passenger contact information
- **Baggage Tracking**: Link baggage to passengers and flights

### 3. Crew Management
- **Crew Roster**: View all pilots and attendants
- **Assignment Dashboard**: Assign crew to flights based on:
  - Aircraft certifications (pilots)
  - Rank requirements
  - Home airport proximity
- **Schedule Calendar**: Visual crew schedule with availability
- **Certification Tracking**: Pilot aircraft certifications

### 4. Booking System
- **Flight Search**: Filter by date, origin, destination
- **Seat Selection**: Interactive seat map with class filtering
- **Fare Calculator**: Real-time pricing based on:
  - Seat class
  - Flight date
  - Fare restrictions
- **Booking Confirmation**: Summary and payment processing

### 5. Airport Operations
- **Airport Dashboard**: Overview of all airports in network
- **Gate Management**: Track gate availability and assignments
- **Flight Board**: Arrivals and departures per airport
- **Ground Staff**: Assign and manage ground crew

### 6. Analytics & Reporting
- **Revenue Dashboard**: Flight profitability by route
- **Occupancy Rates**: Seat utilization metrics
- **Crew Utilization**: Hours flown per crew member
- **Popular Routes**: Most booked flight paths

## UI/UX Design Patterns

### shadcn/ui Components Used

- **Data Display**: Table, Card, Badge, Avatar
- **Forms**: Input, Select, DatePicker, Checkbox, RadioGroup
- **Navigation**: Tabs, NavigationMenu, Breadcrumb
- **Feedback**: Toast, Alert, Dialog, Sheet
- **Layout**: Separator, ScrollArea, Accordion

### Color Scheme (Tailwind)

```javascript
// Primary brand colors
primary: 'hsl(221.2, 83.2%, 53.3%)', // Blue for airline theme
secondary: 'hsl(210, 40%, 96.1%)', // Light blue-gray
accent: 'hsl(210, 40%, 96.1%)',

// Status colors
success: 'hsl(142, 76%, 36%)', // On-time flights
warning: 'hsl(48, 96%, 53%)', // Delayed flights
destructive: 'hsl(0, 84%, 60%)', // Cancelled flights
```

### Responsive Design

- **Mobile**: Stacked cards, hamburger menu, simplified tables
- **Tablet**: Side-by-side panels, collapsible sidebar
- **Desktop**: Full dashboard with multi-column layouts

## Performance Optimizations

### React Query Configuration

```typescript
// app/providers.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      cacheTime: 5 * 60 * 1000, // 5 minutes
      refetchOnWindowFocus: false,
      retry: 1,
    },
  },
});
```

### Optimistic Updates

```typescript
export function useUpdateFlight() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateFlight,
    onMutate: async (newFlight) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['flights'] });

      // Snapshot previous value
      const previousFlights = queryClient.getQueryData(['flights']);

      // Optimistically update
      queryClient.setQueryData(['flights'], (old) => [...old, newFlight]);

      return { previousFlights };
    },
    onError: (err, newFlight, context) => {
      // Rollback on error
      queryClient.setQueryData(['flights'], context.previousFlights);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['flights'] });
    },
  });
}
```

### Table Virtualization

For large datasets (1000+ rows), use `@tanstack/react-virtual`:

```typescript
import { useVirtualizer } from '@tanstack/react-virtual';

// In FlightTable component
const rowVirtualizer = useVirtualizer({
  count: table.getRowModel().rows.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 50,
  overscan: 10,
});
```

## Development Workflow

### Setup Commands

```bash
# Install dependencies
npm install

# Set up database
npm run db:generate  # Generate migrations
npm run db:migrate   # Run migrations
npm run db:push      # Push schema changes
npm run db:studio    # Open Drizzle Studio

# Development
npm run dev          # Start dev server

# Build
npm run build        # Production build
npm run start        # Start production server

# Linting
npm run lint         # ESLint
npm run type-check   # TypeScript check
```

### Environment Variables

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/anchor_air"

# App
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

## Testing Strategy

### Unit Tests
- React Query hooks with mock data
- Form validation schemas
- Utility functions

### Integration Tests
- API route handlers with test database
- Complete booking flow
- Crew assignment logic

### E2E Tests
- Critical user journeys (search → book → confirm)
- Admin operations (create flight, assign crew)

## Future Enhancements

- **Real-time Updates**: WebSocket integration for live flight status
- **Mobile App**: React Native version using shared API
- **Internationalization**: Multi-language support
- **Advanced Analytics**: Data visualization with Recharts/Tremor
- **Email Notifications**: Booking confirmations and flight updates
- **Payment Integration**: Stripe/PayPal for ticket purchases
- **Check-in System**: Online check-in with boarding pass generation
- **Loyalty Program**: Frequent flyer points tracking
