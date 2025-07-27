# VendorBridge - Connect Indian Street Food Vendors & Suppliers

A mobile-first web application connecting Indian street-food vendors with verified suppliers, featuring AI-powered assistance, real-time updates, and comprehensive business management tools.

## 🚀 Features

### Core Features
- **Schedule Management** - Vendors can add/edit/delete required items with quantities and purchase dates
- **Real-time Messaging** - Two-way chat between vendors and suppliers with conversation history
- **Inventory Sync** - Automatic stock deduction and validation with real-time updates
- **Shop Timings** - Supplier open/close times display in marketplace listings
- **Product Photos** - Multiple image support with thumbnails for products
- **Payment Integration** - Razorpay integration with COD as default option

### AI-Powered Features
- **AVA (Automated Vendor Assistant)** - Natural language AI assistant using Google Gemini API
- **SAM (Supply & Alert Monitor)** - Automated pattern analysis and targeted notifications
- **LEO (Logistics & Efficiency Optimizer)** - Intelligent order dispatch and delivery optimization

### Real-time Features
- Live inventory updates
- Real-time messaging
- Push notifications
- Live order tracking
- Instant supplier notifications

## 🛠 Tech Stack

- **Frontend**: Next.js 14 + React + TypeScript + Tailwind CSS + Shadcn UI
- **Backend**: Supabase (Auth, PostgreSQL, Realtime)
- **APIs**: Razorpay, Google Gemini, OpenAI GPT
- **State Management**: React Query
- **Deployment**: Vercel
- **Notifications**: Firebase Cloud Messaging (FCM)

## 📋 Prerequisites

- Node.js 18+ 
- npm or yarn
- Supabase account
- Google Gemini API key
- OpenAI API key (optional)
- Razorpay account (for payments)
- Firebase project (for notifications)

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/vendorbridge.git
cd vendorbridge
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Setup
Create a `.env.local` file in the root directory:

```env
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url_here
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key_here

# AI APIs
GOOGLE_GEMINI_API_KEY=your_gemini_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Payment Gateway
RAZORPAY_KEY_ID=your_razorpay_key_id_here
RAZORPAY_KEY_SECRET=your_razorpay_secret_here

# Firebase (for push notifications)
FIREBASE_PROJECT_ID=your_firebase_project_id_here
FIREBASE_PRIVATE_KEY=your_firebase_private_key_here
FIREBASE_CLIENT_EMAIL=your_firebase_client_email_here

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXTAUTH_SECRET=your_nextauth_secret_here
NEXTAUTH_URL=http://localhost:3000
```

### 4. Database Setup

#### Supabase Setup
1. Create a new Supabase project
2. Run the following SQL to create the database schema:

```sql
-- Enable necessary extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Profiles table
CREATE TABLE profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  avatar_url TEXT,
  role TEXT CHECK (role IN ('vendor', 'supplier', 'admin')) NOT NULL,
  phone TEXT,
  address TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Products table
CREATE TABLE products (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  supplier_id UUID REFERENCES profiles(id) NOT NULL,
  name TEXT NOT NULL,
  description TEXT NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  stock_quantity INTEGER NOT NULL DEFAULT 0,
  category TEXT NOT NULL,
  image_urls TEXT[] DEFAULT '{}',
  unit TEXT NOT NULL,
  min_order_quantity INTEGER DEFAULT 1,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Schedules table
CREATE TABLE schedules (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  vendor_id UUID REFERENCES profiles(id) NOT NULL,
  item_name TEXT NOT NULL,
  quantity INTEGER NOT NULL,
  scheduled_date DATE NOT NULL,
  status TEXT CHECK (status IN ('pending', 'confirmed', 'completed', 'cancelled')) DEFAULT 'pending',
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Orders table
CREATE TABLE orders (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  vendor_id UUID REFERENCES profiles(id) NOT NULL,
  supplier_id UUID REFERENCES profiles(id) NOT NULL,
  product_id UUID REFERENCES products(id) NOT NULL,
  quantity INTEGER NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  status TEXT CHECK (status IN ('pending', 'confirmed', 'shipped', 'delivered', 'cancelled')) DEFAULT 'pending',
  payment_status TEXT CHECK (payment_status IN ('pending', 'paid', 'failed')) DEFAULT 'pending',
  payment_method TEXT CHECK (payment_method IN ('cod', 'online')) DEFAULT 'cod',
  delivery_address TEXT NOT NULL,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Conversations table
CREATE TABLE conversations (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  vendor_id UUID REFERENCES profiles(id) NOT NULL,
  supplier_id UUID REFERENCES profiles(id) NOT NULL,
  last_message TEXT,
  last_message_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(vendor_id, supplier_id)
);

-- Messages table
CREATE TABLE messages (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  conversation_id UUID REFERENCES conversations(id) NOT NULL,
  sender_id UUID REFERENCES profiles(id) NOT NULL,
  receiver_id UUID REFERENCES profiles(id) NOT NULL,
  message TEXT NOT NULL,
  message_type TEXT CHECK (message_type IN ('text', 'image', 'file')) DEFAULT 'text',
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Supplier profiles table
CREATE TABLE supplier_profiles (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  supplier_id UUID REFERENCES profiles(id) NOT NULL,
  shop_name TEXT NOT NULL,
  description TEXT,
  open_time TIME NOT NULL,
  close_time TIME NOT NULL,
  address TEXT NOT NULL,
  phone TEXT NOT NULL,
  rating DECIMAL(3,2) DEFAULT 0,
  total_orders INTEGER DEFAULT 0,
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Alerts table
CREATE TABLE alerts (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  message TEXT NOT NULL,
  priority TEXT CHECK (priority IN ('LOW', 'MEDIUM', 'HIGH', 'CRITICAL', 'URGENT', 'INFO')) NOT NULL,
  data JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Delivery assignments table
CREATE TABLE delivery_assignments (
  id UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  order_id UUID REFERENCES orders(id) NOT NULL,
  rider_id TEXT NOT NULL,
  estimated_pickup_time TIMESTAMP WITH TIME ZONE NOT NULL,
  estimated_delivery_time TIMESTAMP WITH TIME ZONE NOT NULL,
  route_optimization JSONB,
  bundled_orders TEXT[],
  status TEXT CHECK (status IN ('assigned', 'picked_up', 'delivered', 'cancelled')) DEFAULT 'assigned',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE schedules ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE conversations ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE supplier_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE alerts ENABLE ROW LEVEL SECURITY;
ALTER TABLE delivery_assignments ENABLE ROW LEVEL SECURITY;

-- Create RLS policies (basic examples)
CREATE POLICY "Users can view their own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

CREATE POLICY "Anyone can view products" ON products
  FOR SELECT USING (true);

CREATE POLICY "Suppliers can manage their products" ON products
  FOR ALL USING (
    auth.uid() = supplier_id AND 
    EXISTS (SELECT 1 FROM profiles WHERE id = auth.uid() AND role = 'supplier')
  );
```

### 5. Run the Development Server
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the application.

## 📱 Application Structure

```
src/
├── app/                    # Next.js App Router
│   ├── vendor/            # Vendor-specific pages
│   │   ├── dashboard/     # Vendor dashboard
│   │   ├── schedule/      # Schedule management
│   │   ├── marketplace/   # Product browsing
│   │   ├── messages/      # Messaging system
│   │   └── orders/        # Order management
│   ├── supplier/          # Supplier-specific pages
│   │   ├── dashboard/     # Supplier dashboard
│   │   ├── products/      # Product management
│   │   ├── orders/        # Order management
│   │   ├── messages/      # Messaging system
│   │   └── settings/      # Shop settings
│   └── api/               # API routes
├── components/            # Reusable components
│   ├── ui/               # Shadcn UI components
│   └── MessagingSystem.tsx
├── contexts/             # React contexts
│   └── AuthContext.tsx
├── lib/                  # Utility libraries
│   ├── supabase.ts      # Supabase client
│   ├── utils.ts         # Utility functions
│   └── ai/              # AI features
│       ├── ava.ts       # Automated Vendor Assistant
│       ├── sam.ts       # Supply & Alert Monitor
│       └── leo.ts       # Logistics Optimizer
└── providers/           # React providers
    └── QueryProvider.tsx
```

## 🚀 Deployment

### Vercel Deployment
1. Push your code to GitHub
2. Connect your repository to Vercel
3. Add environment variables in Vercel dashboard
4. Deploy!

### Netlify Deployment
1. Push your code to GitHub
2. Connect your repository to Netlify
3. Add environment variables in Netlify dashboard
4. Set build command: `npm run build`
5. Set publish directory: `.next`
6. Deploy!

## 🔧 Configuration

### Environment Variables
All required environment variables are listed in the `.env.local` example above. Make sure to set these in your deployment platform.

### Database Configuration
The application uses Supabase as the backend. Follow the database setup instructions above to create the necessary tables and policies.

### AI Configuration
- **Google Gemini API**: Used for AVA (Automated Vendor Assistant)
- **OpenAI API**: Used for additional AI features (optional)
- **Firebase**: Used for push notifications

## 📊 Features in Detail

### Schedule Management
- Add, edit, and delete purchase schedules
- Set quantities and delivery dates
- Form validation and error handling
- Status tracking (pending, confirmed, completed, cancelled)

### Real-time Messaging
- Two-way chat between vendors and suppliers
- Message history persistence
- Real-time updates using Supabase Realtime
- Read receipts and typing indicators

### AI Features
- **AVA**: Natural language processing for vendor assistance
- **SAM**: Automated supply pattern analysis and alerts
- **LEO**: Intelligent delivery optimization and rider assignment

### Inventory Management
- Real-time stock updates
- Automatic quantity deduction on orders
- Low stock alerts
- Stock validation

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

For support, email support@vendorbridge.com or create an issue in the GitHub repository.

## 🔮 Roadmap

- [ ] Mobile app development
- [ ] Advanced analytics dashboard
- [ ] Multi-language support
- [ ] Advanced payment options
- [ ] Integration with more suppliers
- [ ] Machine learning for demand prediction
- [ ] Blockchain integration for transparency

---

Built with ❤️ for Indian street food vendors and suppliers
