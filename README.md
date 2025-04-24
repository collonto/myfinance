import { useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";

const mockData = [
  {
    username: "bossii",
    role: "admin",
    password: "0010",
  },
  {
    username: "fred",
    role: "customer",
    password: "0000",
    products: [
      { name: "TV", price: 30000, time: "2025-04-24", status: "pending" },
      { name: "Speaker", price: 5000, time: "2025-04-22", status: "completed" },
    ],
  },
  {
    username: "james",
    role: "customer",
    password: "1234",
    products: [
      { name: "Fridge", price: 45000, time: "2025-04-23", status: "pending" },
    ],
  },
];

export default function DominionApp() {
  const [user, setUser] = useState(mockData[0]); // auto-login as admin
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [newProduct, setNewProduct] = useState({ name: "", price: "", time: "", customer: "" });
  const [newCustomer, setNewCustomer] = useState({ username: "", password: "" });

  const handleLogin = () => {
    const found = mockData.find((u) => u.username === username && u.password === password);
    if (found) setUser(found);
    else alert("Wrong credentials");
  };

  const handleAdd = () => {
    const customer = mockData.find((u) => u.username === newProduct.customer);
    if (customer && customer.role === "customer") {
      customer.products.push({
        name: newProduct.name,
        price: parseFloat(newProduct.price),
        time: newProduct.time,
        status: "pending",
      });
      setNewProduct({ name: "", price: "", time: "", customer: "" });
      alert("Product added");
    } else {
      alert("Customer not found");
    }
  };

  const handleDeleteCustomer = (username) => {
    const index = mockData.findIndex((u) => u.username === username);
    if (index !== -1) {
      mockData.splice(index, 1);
      alert("Customer deleted");
    }
  };

  const handleEditCustomerPassword = (username) => {
    const newPass = prompt("Enter new password:");
    if (newPass) {
      const cust = mockData.find((u) => u.username === username);
      if (cust) {
        cust.password = newPass;
        alert("Password updated");
      }
    }
  };

  const calculateTotalPending = (products) => {
    return products.filter(p => p.status === "pending").reduce((sum, p) => sum + p.price, 0);
  };

  if (!user) {
    return (
      <div className="min-h-screen flex flex-col items-center justify-center bg-green-50">
        <h1 className="text-3xl font-bold text-green-800 mb-4">Dominion Login</h1>
        <div className="space-y-2">
          <Input placeholder="Username" value={username} onChange={(e) => setUsername(e.target.value)} />
          <Input placeholder="Password" type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
          <Button onClick={handleLogin} className="bg-green-600 text-white w-full hover:bg-green-700">Login</Button>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen p-4 bg-green-100 text-green-900">
      <h1 className="text-2xl font-bold text-center mb-6">Welcome to Dominion, {user.username}</h1>
      {user.role === "admin" ? (
        <>
          <Card className="max-w-xl mx-auto mb-6">
            <CardContent className="space-y-4">
              <h2 className="text-xl font-semibold">Add Product (Credit)</h2>
              <Input placeholder="Product Name" value={newProduct.name} onChange={(e) => setNewProduct({ ...newProduct, name: e.target.value })} />
              <Input placeholder="Price" type="number" value={newProduct.price} onChange={(e) => setNewProduct({ ...newProduct, price: e.target.value })} />
              <Input placeholder="Date" type="date" value={newProduct.time} onChange={(e) => setNewProduct({ ...newProduct, time: e.target.value })} />
              <Input placeholder="Customer Username" value={newProduct.customer} onChange={(e) => setNewProduct({ ...newProduct, customer: e.target.value })} />
              <Button onClick={handleAdd} className="bg-green-600 text-white hover:bg-green-700 w-full">Add</Button>
            </CardContent>
          </Card>

          <Card className="max-w-xl mx-auto mb-6">
            <CardContent className="space-y-4">
              <h2 className="text-xl font-semibold">Add New Customer</h2>
              <Input placeholder="Customer Username" value={newCustomer.username} onChange={(e) => setNewCustomer({ ...newCustomer, username: e.target.value })} />
              <Input placeholder="Password" type="password" value={newCustomer.password} onChange={(e) => setNewCustomer({ ...newCustomer, password: e.target.value })} />
              <Button onClick={() => {
                if (!newCustomer.username || !newCustomer.password) return alert("Fill in all fields");
                mockData.push({
                  username: newCustomer.username,
                  password: newCustomer.password,
                  role: "customer",
                  products: [],
                });
                setNewCustomer({ username: "", password: "" });
                alert("Customer added");
              }} className="bg-green-600 text-white hover:bg-green-700 w-full">Add Customer</Button>
            </CardContent>
          </Card>

          <Card className="max-w-xl mx-auto">
            <CardContent className="space-y-4">
              <h2 className="text-xl font-semibold">Customer Portfolios</h2>
              {mockData.filter(u => u.role === "customer").map((cust, idx) => (
                <div key={idx} className="border-t pt-2">
                  <h3 className="font-bold flex justify-between items-center">
                    {cust.username}
                    <span className="text-sm space-x-2">
                      <Button size="sm" variant="outline" onClick={() => handleEditCustomerPassword(cust.username)}>Edit</Button>
                      <Button size="sm" variant="destructive" onClick={() => handleDeleteCustomer(cust.username)}>Delete</Button>
                    </span>
                  </h3>
                  {cust.products.map((p, i) => (
                    <div key={i} className="border-b py-1">
                      <p className="text-sm">{p.name} - KES {p.price.toLocaleString()} on {p.time}</p>
                      <p className={`text-sm ${p.status === "completed" ? "text-green-600" : "text-red-500"}`}>{p.status.toUpperCase()}</p>
                    </div>
                  ))}
                  <p className="text-right text-sm font-semibold mt-1">Total Pending: KES {calculateTotalPending(cust.products).toLocaleString()}</p>
                </div>
              ))}
            </CardContent>
          </Card>
        </>
      ) : (
        <Card className="max-w-xl mx-auto">
          <CardContent className="space-y-4">
            <h2 className="text-xl font-semibold">Your Credit Products</h2>
            {user.products.map((item, index) => (
              <div key={index} className="border-b py-2">
                <p className="font-medium">{item.name}</p>
                <p className="text-sm">KES {item.price.toLocaleString()} on {item.time}</p>
                <p className={`text-sm font-semibold ${item.status === "completed" ? "text-green-600" : "text-red-500"}`}>{item.status.toUpperCase()}</p>
              </div>
            ))}
            <Button className="w-full bg-blue-600 hover:bg-blue-700 text-white">Send M-Pesa Prompt</Button>
          </CardContent>
        </Card>
      )}
    </div>
  );
}
