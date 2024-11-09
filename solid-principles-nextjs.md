# SOLID Principles in Next.js: A Practical Guide

## 1. Single Responsibility Principle (SRP)
Each class or component should have only one reason to change.

### Bad Example:
```typescript
// pages/UserProfile.tsx
const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [error, setError] = useState('');

  const fetchUserData = async () => {
    try {
      const response = await fetch('/api/user');
      const data = await response.json();
      setUser(data);
    } catch (err) {
      setError('Failed to fetch user');
    }
  };

  const updateUserProfile = async (userData) => {
    try {
      await fetch('/api/user', {
        method: 'PUT',
        body: JSON.stringify(userData)
      });
    } catch (err) {
      setError('Failed to update user');
    }
  };

  return (
    <div>
      {error && <div>{error}</div>}
      {user && (
        <div>
          <h1>{user.name}</h1>
          <p>{user.email}</p>
          {/* More user information */}
        </div>
      )}
    </div>
  );
};
```

### Good Example:
```typescript
// services/userService.ts
export const userService = {
  async fetchUser() {
    const response = await fetch('/api/user');
    return response.json();
  },
  
  async updateUser(userData) {
    const response = await fetch('/api/user', {
      method: 'PUT',
      body: JSON.stringify(userData)
    });
    return response.json();
  }
};

// components/UserProfileView.tsx
const UserProfileView = ({ user }) => (
  <div>
    <h1>{user.name}</h1>
    <p>{user.email}</p>
    {/* More user information */}
  </div>
);

// pages/UserProfile.tsx
const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [error, setError] = useState('');

  useEffect(() => {
    const loadUser = async () => {
      try {
        const data = await userService.fetchUser();
        setUser(data);
      } catch (err) {
        setError('Failed to fetch user');
      }
    };
    loadUser();
  }, []);

  return (
    <div>
      {error && <div>{error}</div>}
      {user && <UserProfileView user={user} />}
    </div>
  );
};
```

## 2. Open-Closed Principle (OCP)
Software entities should be open for extension but closed for modification.

### Bad Example:
```typescript
// components/Button.tsx
const Button = ({ type, ...props }) => {
  if (type === 'primary') {
    return <button className="bg-blue-500" {...props} />;
  } else if (type === 'secondary') {
    return <button className="bg-gray-500" {...props} />;
  } else if (type === 'danger') {
    return <button className="bg-red-500" {...props} />;
  }
  return <button {...props} />;
};
```

### Good Example:
```typescript
// components/Button.tsx
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  className?: string;
  [key: string]: any;
}

const buttonVariants = {
  primary: 'bg-blue-500 hover:bg-blue-600',
  secondary: 'bg-gray-500 hover:bg-gray-600',
  danger: 'bg-red-500 hover:bg-red-600',
};

const Button = ({ variant = 'primary', className = '', ...props }: ButtonProps) => (
  <button 
    className={`${buttonVariants[variant]} ${className}`}
    {...props}
  />
);
```

## 3. Liskov Substitution Principle (LSP)
Derived classes must be substitutable for their base classes.

### Bad Example:
```typescript
// components/Card.tsx
class BaseCard {
  render() {
    return <div className="p-4 border rounded" />;
  }
}

class ProductCard extends BaseCard {
  render() {
    // Breaking LSP by requiring specific props not present in base class
    if (!this.props.product) {
      throw new Error('Product is required');
    }
    return <div className="p-4 border rounded">{this.props.product.name}</div>;
  }
}
```

### Good Example:
```typescript
// components/Card.tsx
interface CardProps {
  children: React.ReactNode;
  className?: string;
}

const Card = ({ children, className = '' }: CardProps) => (
  <div className={`p-4 border rounded ${className}`}>
    {children}
  </div>
);

interface ProductCardProps {
  product: {
    name: string;
    price: number;
  };
}

const ProductCard = ({ product }: ProductCardProps) => (
  <Card className="product-card">
    <h3>{product.name}</h3>
    <p>${product.price}</p>
  </Card>
);
```

## 4. Interface Segregation Principle (ISP)
Clients should not be forced to depend on interfaces they do not use.

### Bad Example:
```typescript
// types/user.ts
interface User {
  id: string;
  name: string;
  email: string;
  address: string;
  paymentMethods: PaymentMethod[];
  orders: Order[];
  preferences: UserPreferences;
}

// components/UserHeader.tsx
const UserHeader = ({ user }: { user: User }) => (
  <div>
    <h1>{user.name}</h1>
    <p>{user.email}</p>
  </div>
);
```

### Good Example:
```typescript
// types/user.ts
interface UserBasicInfo {
  id: string;
  name: string;
  email: string;
}

interface UserAddress {
  address: string;
}

interface UserPayment {
  paymentMethods: PaymentMethod[];
}

interface UserOrders {
  orders: Order[];
}

interface UserPreferences {
  preferences: Preferences;
}

// Use type intersection if needed
type FullUser = UserBasicInfo & UserAddress & UserPayment & UserOrders & UserPreferences;

// components/UserHeader.tsx
const UserHeader = ({ user }: { user: UserBasicInfo }) => (
  <div>
    <h1>{user.name}</h1>
    <p>{user.email}</p>
  </div>
);
```

## 5. Dependency Inversion Principle (DIP)
High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Bad Example:
```typescript
// pages/products.tsx
const ProductsPage = () => {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    const fetchProducts = async () => {
      const response = await fetch('/api/products');
      const data = await response.json();
      setProducts(data);
    };
    fetchProducts();
  }, []);

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
};
```

### Good Example:
```typescript
// services/api/types.ts
interface ApiClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
}

// services/api/httpClient.ts
export class HttpClient implements ApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    return response.json();
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(url, {
      method: 'POST',
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

// services/products/types.ts
interface ProductsRepository {
  getProducts(): Promise<Product[]>;
}

// services/products/productsRepository.ts
export class ApiProductsRepository implements ProductsRepository {
  constructor(private apiClient: ApiClient) {}

  async getProducts(): Promise<Product[]> {
    return this.apiClient.get<Product[]>('/api/products');
  }
}

// hooks/useProducts.ts
export const useProducts = (productsRepository: ProductsRepository) => {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchProducts = async () => {
      try {
        setLoading(true);
        const data = await productsRepository.getProducts();
        setProducts(data);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };
    fetchProducts();
  }, [productsRepository]);

  return { products, loading, error };
};

// pages/products.tsx
const ProductsPage = () => {
  const apiClient = new HttpClient();
  const productsRepository = new ApiProductsRepository(apiClient);
  const { products, loading, error } = useProducts(productsRepository);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
};
```

## Best Practices and Tips

1. Use TypeScript to enforce interfaces and type safety
2. Create small, focused components
3. Implement dependency injection where possible
4. Use custom hooks to separate business logic
5. Create service layers for API calls and business logic
6. Use interfaces to define contracts between components
7. Implement error boundaries for better error handling
8. Use composition over inheritance

## Conclusion

Implementing SOLID principles in Next.js applications leads to:
- More maintainable code
- Better testability
- Easier to extend functionality
- Reduced coupling between components
- More scalable architecture

Remember that SOLID principles are guidelines, not strict rules. Apply them where they make sense for your specific use case and team size.