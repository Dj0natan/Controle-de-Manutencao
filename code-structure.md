# Sistema de Gestão - Estrutura Completa do Código

## Estrutura de Arquivos

```
projeto/
├── client/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ui/           # Componentes UI shadcn/ui
│   │   │   ├── customer-modal.tsx
│   │   │   ├── employee-modal.tsx
│   │   │   ├── service-modal.tsx
│   │   │   └── sidebar.tsx
│   │   ├── pages/
│   │   │   ├── dashboard.tsx
│   │   │   ├── customers.tsx
│   │   │   ├── employees.tsx
│   │   │   ├── services.tsx
│   │   │   └── not-found.tsx
│   │   ├── hooks/
│   │   │   ├── use-mobile.tsx
│   │   │   └── use-toast.ts
│   │   ├── lib/
│   │   │   ├── queryClient.ts
│   │   │   ├── utils.ts
│   │   │   └── cpf-validator.ts
│   │   ├── App.tsx
│   │   ├── index.css
│   │   └── main.tsx
│   └── index.html
├── server/
│   ├── index.ts
│   ├── routes.ts
│   ├── storage.ts
│   └── vite.ts
├── shared/
│   └── schema.ts
└── arquivos de configuração...
```

## Principais Arquivos

### 1. Schema de Dados (shared/schema.ts)
```typescript
import { pgTable, text, serial, integer, boolean, timestamp } from "drizzle-orm/pg-core";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";

export const customers = pgTable("customers", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  document: text("document").notNull(),
  email: text("email").notNull(),
  phone: text("phone").notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

export const employees = pgTable("employees", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  position: text("position").notNull(),
  email: text("email").notNull(),
  phone: text("phone").notNull(),
  status: text("status").notNull().default("active"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

export const services = pgTable("services", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  description: text("description").notNull(),
  estimatedTime: integer("estimated_time").notNull(),
  timeUnit: text("time_unit").notNull().default("hours"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

// Schemas de validação
export const insertCustomerSchema = createInsertSchema(customers).omit({
  id: true,
  createdAt: true,
});

export const insertEmployeeSchema = createInsertSchema(employees).omit({
  id: true,
  createdAt: true,
});

export const insertServiceSchema = createInsertSchema(services).omit({
  id: true,
  createdAt: true,
});

// Tipos TypeScript
export type Customer = typeof customers.$inferSelect;
export type Employee = typeof employees.$inferSelect;
export type Service = typeof services.$inferSelect;
export type InsertCustomer = z.infer<typeof insertCustomerSchema>;
export type InsertEmployee = z.infer<typeof insertEmployeeSchema>;
export type InsertService = z.infer<typeof insertServiceSchema>;
```

### 2. Armazenamento em Memória (server/storage.ts)
```typescript
import { 
  type Customer, type InsertCustomer,
  type Employee, type InsertEmployee,
  type Service, type InsertService
} from "@shared/schema";

export interface IStorage {
  // Customers
  getAllCustomers(): Promise<Customer[]>;
  createCustomer(customer: InsertCustomer): Promise<Customer>;
  updateCustomer(id: number, customer: Partial<InsertCustomer>): Promise<Customer | undefined>;
  deleteCustomer(id: number): Promise<boolean>;
  
  // Employees
  getAllEmployees(): Promise<Employee[]>;
  createEmployee(employee: InsertEmployee): Promise<Employee>;
  updateEmployee(id: number, employee: Partial<InsertEmployee>): Promise<Employee | undefined>;
  deleteEmployee(id: number): Promise<boolean>;
  
  // Services
  getAllServices(): Promise<Service[]>;
  createService(service: InsertService): Promise<Service>;
  updateService(id: number, service: Partial<InsertService>): Promise<Service | undefined>;
  deleteService(id: number): Promise<boolean>;
}

export class MemStorage implements IStorage {
  private customers: Map<number, Customer> = new Map();
  private employees: Map<number, Employee> = new Map();
  private services: Map<number, Service> = new Map();
  private currentCustomerId: number = 1;
  private currentEmployeeId: number = 1;
  private currentServiceId: number = 1;

  // Implementação de todos os métodos CRUD...
}

export const storage = new MemStorage();
```

### 3. Rotas da API (server/routes.ts)
```typescript
import { storage } from "./storage";
import { insertCustomerSchema, insertEmployeeSchema, insertServiceSchema } from "@shared/schema";

export async function registerRoutes(app: Express): Promise<Server> {
  // Rotas de clientes
  app.get("/api/customers", async (req, res) => {
    const customers = await storage.getAllCustomers();
    res.json(customers);
  });

  app.post("/api/customers", async (req, res) => {
    const validatedData = insertCustomerSchema.parse(req.body);
    const customer = await storage.createCustomer(validatedData);
    res.status(201).json(customer);
  });

  // Rotas similares para funcionários e serviços...
  
  // Endpoint de estatísticas
  app.get("/api/stats", async (req, res) => {
    const customers = await storage.getAllCustomers();
    const employees = await storage.getAllEmployees();
    const services = await storage.getAllServices();
    
    res.json({
      customers: customers.length,
      employees: employees.length,
      services: services.length,
    });
  });
}
```

### 4. Componente Principal (client/src/App.tsx)
```typescript
import { Switch, Route } from "wouter";
import { QueryClientProvider } from "@tanstack/react-query";
import { queryClient } from "./lib/queryClient";
import Dashboard from "@/pages/dashboard";
import Customers from "@/pages/customers";
import Employees from "@/pages/employees";
import Services from "@/pages/services";
import Sidebar from "@/components/sidebar";

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="flex h-screen bg-slate-50">
        <Sidebar />
        <div className="flex-1 overflow-auto">
          <Switch>
            <Route path="/" component={Dashboard} />
            <Route path="/customers" component={Customers} />
            <Route path="/employees" component={Employees} />
            <Route path="/services" component={Services} />
          </Switch>
        </div>
      </div>
    </QueryClientProvider>
  );
}
```

### 5. Dashboard (client/src/pages/dashboard.tsx)
```typescript
import { useQuery } from "@tanstack/react-query";
import { Card, CardContent } from "@/components/ui/card";
import { Users, UserCheck, Settings } from "lucide-react";

export default function Dashboard() {
  const { data: stats } = useQuery({
    queryKey: ["/api/stats"],
  });

  return (
    <div className="p-6">
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <Card>
          <CardContent className="p-6">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-slate-600">Total de Clientes</p>
                <p className="text-2xl font-semibold">{stats?.customers ?? 0}</p>
              </div>
              <Users className="text-blue-600" size={20} />
            </div>
          </CardContent>
        </Card>
        
        <Card>
          <CardContent className="p-6">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-slate-600">Funcionários Ativos</p>
                <p className="text-2xl font-semibold">{stats?.employees ?? 0}</p>
              </div>
              <UserCheck className="text-green-600" size={20} />
            </div>
          </CardContent>
        </Card>
        
        <Card>
          <CardContent className="p-6">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-slate-600">Serviços Cadastrados</p>
                <p className="text-2xl font-semibold">{stats?.services ?? 0}</p>
              </div>
              <Settings className="text-purple-600" size={20} />
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

### 6. Validação de CPF/CNPJ (client/src/lib/cpf-validator.ts)
```typescript
export function validateCPF(cpf: string): boolean {
  const cleaned = cpf.replace(/\D/g, '');
  
  if (cleaned.length !== 11) return false;
  if (/^(\d)\1+$/.test(cleaned)) return false;
  
  // Validação dos dígitos verificadores
  let sum = 0;
  for (let i = 0; i < 9; i++) {
    sum += parseInt(cleaned[i]) * (10 - i);
  }
  let remainder = sum % 11;
  let firstCheckDigit = remainder < 2 ? 0 : 11 - remainder;
  
  if (parseInt(cleaned[9]) !== firstCheckDigit) return false;
  
  sum = 0;
  for (let i = 0; i < 10; i++) {
    sum += parseInt(cleaned[i]) * (11 - i);
  }
  remainder = sum % 11;
  let secondCheckDigit = remainder < 2 ? 0 : 11 - remainder;
  
  return parseInt(cleaned[10]) === secondCheckDigit;
}

export function validateCNPJ(cnpj: string): boolean {
  // Implementação similar para CNPJ...
}
```

### 7. Modal de Cliente (client/src/components/customer-modal.tsx)
```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { validateCPF, validateCNPJ } from "@/lib/cpf-validator";

const customerFormSchema = insertCustomerSchema.extend({
  document: z.string().min(1, "Documento é obrigatório").refine((val) => {
    const cleaned = val.replace(/\D/g, '');
    return validateCPF(cleaned) || validateCNPJ(cleaned);
  }, "CPF ou CNPJ inválido"),
  email: z.string().email("E-mail inválido"),
  phone: z.string().min(10, "Telefone deve ter pelo menos 10 dígitos"),
});

export default function CustomerModal({ isOpen, onClose, customer }) {
  const form = useForm({
    resolver: zodResolver(customerFormSchema),
    defaultValues: {
      name: "",
      document: "",
      email: "",
      phone: "",
    },
  });

  const createMutation = useMutation({
    mutationFn: async (data) => {
      const response = await apiRequest("POST", "/api/customers", data);
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["/api/customers"] });
      onClose();
    },
  });

  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>
            {customer ? "Editar Cliente" : "Novo Cliente"}
          </DialogTitle>
        </DialogHeader>
        
        <form onSubmit={form.handleSubmit(onSubmit)}>
          <Input {...form.register("name")} placeholder="Nome completo" />
          <Input {...form.register("document")} placeholder="CPF/CNPJ" />
          <Input {...form.register("email")} placeholder="E-mail" />
          <Input {...form.register("phone")} placeholder="Telefone" />
          
          <div className="flex space-x-3 pt-4">
            <Button type="button" variant="outline" onClick={onClose}>
              Cancelar
            </Button>
            <Button type="submit">
              Salvar Cliente
            </Button>
          </div>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

## Funcionalidades Implementadas

### ✅ Módulo de Clientes
- Listagem com tabela responsiva
- Cadastro com validação de CPF/CNPJ
- Edição de dados
- Formatação automática de documentos e telefones
- Validação de e-mail

### ✅ Módulo de Funcionários
- Listagem com status (Ativo/Inativo)
- Cadastro com seleção de cargo
- Gerenciamento de status
- Badges visuais para status

### ✅ Módulo de Serviços
- Listagem em cards
- Cadastro com descrição
- Tempo estimado configurável (horas, dias, semanas)
- Interface visual intuitiva

### ✅ Dashboard
- Estatísticas em tempo real
- Cards informativos
- Atividades recentes
- Design responsivo

### ✅ Navegação
- Sidebar lateral fixa
- Roteamento com wouter
- Indicadores visuais de página ativa
- Interface responsiva

### ✅ Validações
- Formulários com react-hook-form
- Validação com Zod
- Validação brasileira (CPF/CNPJ)
- Formatação automática de campos

### ✅ API REST
- Endpoints para todas as operações CRUD
- Validação de dados no backend
- Tratamento de erros
- Armazenamento em memória

## Tecnologias Utilizadas

- **Frontend**: React, TypeScript, Tailwind CSS, shadcn/ui
- **Backend**: Express.js, TypeScript
- **Validação**: Zod, react-hook-form
- **Estado**: TanStack Query
- **Roteamento**: wouter
- **Banco de Dados**: Armazenamento em memória (MemStorage)
- **Build**: Vite

Este sistema está completo e funcional, seguindo as melhores práticas de desenvolvimento com TypeScript, validação robusta e interface moderna.