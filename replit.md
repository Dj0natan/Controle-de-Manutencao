# Sistema de Gestão

## Overview

This is a full-stack TypeScript application for a business management system built with React (frontend) and Express (backend). The application provides CRUD operations for customers, employees, and services with a modern, responsive UI using shadcn/ui components.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite for fast development and optimized production builds
- **Routing**: wouter for client-side routing
- **State Management**: TanStack Query (React Query) for server state management
- **Styling**: Tailwind CSS with shadcn/ui component library
- **Forms**: React Hook Form with Zod validation
- **UI Components**: Radix UI primitives via shadcn/ui

### Backend Architecture
- **Framework**: Express.js with TypeScript
- **Database**: PostgreSQL with Drizzle ORM
- **Validation**: Zod schemas for API validation
- **Database Provider**: Neon Database (serverless PostgreSQL)
- **Storage**: In-memory storage implementation with interface for easy swapping

## Key Components

### Database Schema
The application uses Drizzle ORM with PostgreSQL and defines four main entities:
- **Users**: Basic user authentication (id, username, password)
- **Customers**: Customer management (id, name, document, email, phone, createdAt)
- **Employees**: Employee management (id, name, position, email, phone, status, createdAt)
- **Services**: Service catalog (id, name, description, estimatedTime, timeUnit, createdAt)

### API Structure
RESTful API endpoints following standard conventions:
- `GET /api/customers` - List all customers
- `POST /api/customers` - Create new customer
- `PUT /api/customers/:id` - Update customer
- Similar patterns for employees and services

### Frontend Pages
- **Dashboard**: Overview with statistics and quick actions
- **Customers**: Customer management with CRUD operations
- **Employees**: Employee management with status tracking
- **Services**: Service catalog management with time estimation
- **Reports**: Analytics and reporting module with export functionality

### Modal Components
Reusable modal components for entity management:
- **CustomerModal**: Form validation includes CPF/CNPJ validation for Brazilian documents
- **EmployeeModal**: Employee creation/editing with position and status management
- **ServiceModal**: Service management with time unit selection

## Data Flow

1. **Client Request**: User interactions trigger API calls via TanStack Query
2. **API Layer**: Express routes validate input using Zod schemas
3. **Storage Layer**: Currently uses in-memory storage implementing IStorage interface
4. **Database Layer**: Drizzle ORM handles PostgreSQL operations (when connected)
5. **Response**: JSON responses with proper error handling and logging

## External Dependencies

### Core Dependencies
- **@neondatabase/serverless**: Serverless PostgreSQL database connection
- **drizzle-orm**: Type-safe database ORM
- **@tanstack/react-query**: Server state management
- **react-hook-form**: Form handling and validation
- **zod**: Schema validation
- **tailwindcss**: Utility-first CSS framework

### UI Dependencies
- **@radix-ui**: Headless UI components for accessibility
- **class-variance-authority**: Utility for component variants
- **lucide-react**: Icon library
- **date-fns**: Date manipulation utilities

## Deployment Strategy

### Development
- **Dev Server**: Uses Vite dev server with Express backend
- **Hot Reload**: Vite HMR for frontend, tsx for backend auto-restart
- **Environment**: NODE_ENV=development with detailed logging

### Production Build
- **Frontend**: Vite builds static assets to `dist/public`
- **Backend**: esbuild bundles Express server to `dist/index.js`
- **Database**: Drizzle migrations applied via `db:push` command
- **Deployment**: Single server deployment with static file serving

### Database Management
- **Migrations**: Stored in `./migrations` directory
- **Schema**: Centralized in `shared/schema.ts`
- **Environment**: Requires `DATABASE_URL` environment variable

The application is structured as a monorepo with shared TypeScript types and schemas, making it easy to maintain consistency between frontend and backend while supporting both development and production environments.

## Recent Changes: Latest modifications with dates

### January 13, 2025
- **Added Reports Module**: Complete analytics and reporting functionality
  - New `/reports` page with overview statistics and detailed analytics
  - Export functionality for CSV reports (customers, employees, services, overview)
  - Report filtering by type and date range
  - Backend endpoint `/api/reports` for detailed analytics data
  - Utility functions for report generation and data formatting
  - Integration with existing CRUD operations for real-time data
  - Professional dashboard layout with cards and tables

- **Enhanced Role-Based Access Control**: Complete implementation of employee position-based permissions
  - Added `role` field to employee management with hierarchy levels (funcionario, tecnico, supervisor, coordenador, gerente, diretor, admin)
  - Role-based UI controls: supervisors+ can create/edit employees, managers+ can delete
  - Visual role indicators with color-coded badges and shield icons
  - Employee table updated to display access levels with proper formatting
  - Enhanced employee modal with role selection dropdown
  - Integrated with existing authentication system for complete access control