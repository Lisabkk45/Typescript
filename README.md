import React, { useState, useEffect } from 'react';
import { LayoutDashboard, Package, Users, ShoppingCart, Plus, Edit2, Trash2, Search } from 'lucide-react';

export default function ERPApp() {
  const [activeMenu, setActiveMenu] = useState('dashboard');
  const [products, setProducts] = useState([]);
  const [customers, setCustomers] = useState([]);
  const [orders, setOrders] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [showModal, setShowModal] = useState(false);
  const [modalType, setModalType] = useState('');
  const [editingItem, setEditingItem] = useState(null);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      const productsData = await window.storage.get('products');
      const customersData = await window.storage.get('customers');
      const ordersData = await window.storage.get('orders');
      
      if (productsData) setProducts(JSON.parse(productsData.value));
      if (customersData) setCustomers(JSON.parse(customersData.value));
      if (ordersData) setOrders(JSON.parse(ordersData.value));
    } catch (error) {
      console.log('No existing data');
    }
  };

  const saveProducts = async (data) => {
    await window.storage.set('products', JSON.stringify(data));
    setProducts(data);
  };

  const saveCustomers = async (data) => {
    await window.storage.set('customers', JSON.stringify(data));
    setCustomers(data);
  };

  const saveOrders = async (data) => {
    await window.storage.set('orders', JSON.stringify(data));
    setOrders(data);
  };

  const handleAddProduct = (formData) => {
    const newProduct = {
      id: Date.now(),
      name: formData.name,
      sku: formData.sku,
      price: parseFloat(formData.price),
      stock: parseInt(formData.stock)
    };
    saveProducts([...products, newProduct]);
  };

  const handleEditProduct = (formData) => {
    const updated = products.map(p => p.id === editingItem.id ? { ...p, ...formData, price: parseFloat(formData.price), stock: parseInt(formData.stock) } : p);
    saveProducts(updated);
  };

  const handleDeleteProduct = (id) => {
    saveProducts(products.filter(p => p.id !== id));
  };

  const handleAddCustomer = (formData) => {
    const newCustomer = {
      id: Date.now(),
      name: formData.name,
      email: formData.email,
      phone: formData.phone,
      company: formData.company
    };
    saveCustomers([...customers, newCustomer]);
  };

  const handleEditCustomer = (formData) => {
    const updated = customers.map(c => c.id === editingItem.id ? { ...c, ...formData } : c);
    saveCustomers(updated);
  };

  const handleDeleteCustomer = (id) => {
    saveCustomers(customers.filter(c => c.id !== id));
  };

  const handleAddOrder = (formData) => {
    const newOrder = {
      id: Date.now(),
      customerId: formData.customerId,
      productId: formData.productId,
      quantity: parseInt(formData.quantity),
      date: new Date().toISOString().split('T')[0],
      status: formData.status
    };
    saveOrders([...orders, newOrder]);
  };

  const handleEditOrder = (formData) => {
    const updated = orders.map(o => o.id === editingItem.id ? { ...o, ...formData, quantity: parseInt(formData.quantity) } : o);
    saveOrders(updated);
  };

  const handleDeleteOrder = (id) => {
    saveOrders(orders.filter(o => o.id !== id));
  };

  const openModal = (type, item = null) => {
    setModalType(type);
    setEditingItem(item);
    setShowModal(true);
  };

  const closeModal = () => {
    setShowModal(false);
    setEditingItem(null);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const data = Object.fromEntries(formData);

    if (modalType === 'product') {
      editingItem ? handleEditProduct(data) : handleAddProduct(data);
    } else if (modalType === 'customer') {
      editingItem ? handleEditCustomer(data) : handleAddCustomer(data);
    } else if (modalType === 'order') {
      editingItem ? handleEditOrder(data) : handleAddOrder(data);
    }
    closeModal();
  };

  const Dashboard = () => {
    const totalRevenue = orders.reduce((sum, order) => {
      const product = products.find(p => p.id === parseInt(order.productId));
      return sum + (product ? product.price * order.quantity : 0);
    }, 0);

    const lowStockProducts = products.filter(p => p.stock < 10);

    return (
      <div className="p-6">
        <h2 className="text-2xl font-bold mb-6">Dashboard</h2>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
          <div className="bg-blue-50 p-6 rounded-lg border border-blue-200">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-blue-600 font-medium">Total Products</p>
                <p className="text-3xl font-bold text-blue-900">{products.length}</p>
              </div>
              <Package className="text-blue-500" size={40} />
            </div>
          </div>
          <div className="bg-green-50 p-6 rounded-lg border border-green-200">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-green-600 font-medium">Total Customers</p>
                <p className="text-3xl font-bold text-green-900">{customers.length}</p>
              </div>
              <Users className="text-green-500" size={40} />
            </div>
          </div>
          <div className="bg-purple-50 p-6 rounded-lg border border-purple-200">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-sm text-purple-600 font-medium">Total Orders</p>
                <p className="text-3xl font-bold text-purple-900">{orders.length}</p>
              </div>
              <ShoppingCart className="text-purple-500" size={40} />
            </div>
          </div>
        </div>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div className="bg-white p-6 rounded-lg border">
            <h3 className="text-lg font-semibold mb-4">Revenue Overview</h3>
            <p className="text-3xl font-bold text-gray-900">${totalRevenue.toFixed(2)}</p>
            <p className="text-sm text-gray-500 mt-2">Total revenue from all orders</p>
          </div>
          <div className="bg-white p-6 rounded-lg border">
            <h3 className="text-lg font-semibold mb-4">Low Stock Alert</h3>
            {lowStockProducts.length > 0 ? (
              <div className="space-y-2">
                {lowStockProducts.map(product => (
                  <div key={product.id} className="flex justify-between items-center text-sm">
                    <span className="text-gray-700">{product.name}</span>
                    <span className="text-red-600 font-medium">{product.stock} units</span>
                  </div>
                ))}
              </div>
            ) : (
              <p className="text-gray-500">All products are well stocked</p>
            )}
          </div>
        </div>
      </div>
    );
  };

  const ProductsView = () => {
    const filtered = products.filter(p => 
      p.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      p.sku.toLowerCase().includes(searchTerm.toLowerCase())
    );

    return (
      <div className="p-6">
        <div className="flex justify-between items-center mb-6">
          <h2 className="text-2xl font-bold">Products</h2>
          <button onClick={() => openModal('product')} className="bg-blue-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-blue-700">
            <Plus size={20} /> Add Product
          </button>
        </div>
        
        <div className="mb-4">
          <div className="relative">
            <Search className="absolute left-3 top-3 text-gray-400" size={20} />
            <input
              type="text"
              placeholder="Search products..."
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
              className="w-full pl-10 pr-4 py-2 border rounded-lg"
            />
          </div>
        </div>

        <div className="bg-white rounded-lg border overflow-hidden">
          <table className="w-full">
            <thead className="bg-gray-50">
              <tr>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Product Name</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">SKU</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Price</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Stock</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y">
              {filtered.map(product => (
                <tr key={product.id} className="hover:bg-gray-50">
                  <td className="px-6 py-4 text-sm text-gray-900">{product.name}</td>
                  <td className="px-6 py-4 text-sm text-gray-600">{product.sku}</td>
                  <td className="px-6 py-4 text-sm text-gray-900">${product.price.toFixed(2)}</td>
                  <td className="px-6 py-4 text-sm text-gray-900">{product.stock}</td>
                  <td className="px-6 py-4 text-sm">
                    <button onClick={() => openModal('product', product)} className="text-blue-600 hover:text-blue-800 mr-3">
                      <Edit2 size={18} />
                    </button>
                    <button onClick={() => handleDeleteProduct(product.id)} className="text-red-600 hover:text-red-800">
                      <Trash2 size={18} />
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
    );
  };

  const CustomersView = () => {
    const filtered = customers.filter(c => 
      c.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      c.email.toLowerCase().includes(searchTerm.toLowerCase())
    );

    return (
      <div className="p-6">
        <div className="flex justify-between items-center mb-6">
          <h2 className="text-2xl font-bold">Customers</h2>
          <button onClick={() => openModal('customer')} className="bg-blue-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-blue-700">
            <Plus size={20} /> Add Customer
          </button>
        </div>
        
        <div className="mb-4">
          <div className="relative">
            <Search className="absolute left-3 top-3 text-gray-400" size={20} />
            <input
              type="text"
              placeholder="Search customers..."
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
              className="w-full pl-10 pr-4 py-2 border rounded-lg"
            />
          </div>
        </div>

        <div className="bg-white rounded-lg border overflow-hidden">
          <table className="w-full">
            <thead className="bg-gray-50">
              <tr>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Name</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Email</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Phone</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Company</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y">
              {filtered.map(customer => (
                <tr key={customer.id} className="hover:bg-gray-50">
                  <td className="px-6 py-4 text-sm text-gray-900">{customer.name}</td>
                  <td className="px-6 py-4 text-sm text-gray-600">{customer.email}</td>
                  <td className="px-6 py-4 text-sm text-gray-600">{customer.phone}</td>
                  <td className="px-6 py-4 text-sm text-gray-600">{customer.company}</td>
                  <td className="px-6 py-4 text-sm">
                    <button onClick={() => openModal('customer', customer)} className="text-blue-600 hover:text-blue-800 mr-3">
                      <Edit2 size={18} />
                    </button>
                    <button onClick={() => handleDeleteCustomer(customer.id)} className="text-red-600 hover:text-red-800">
                      <Trash2 size={18} />
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
    );
  };

  const OrdersView = () => {
    const filtered = orders.filter(o => {
      const customer = customers.find(c => c.id === parseInt(o.customerId));
      const product = products.find(p => p.id === parseInt(o.productId));
      return customer?.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
             product?.name.toLowerCase().includes(searchTerm.toLowerCase());
    });

    return (
      <div className="p-6">
        <div className="flex justify-between items-center mb-6">
          <h2 className="text-2xl font-bold">Orders</h2>
          <button onClick={() => openModal('order')} className="bg-blue-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-blue-700">
            <Plus size={20} /> Add Order
          </button>
        </div>
        
        <div className="mb-4">
          <div className="relative">
            <Search className="absolute left-3 top-3 text-gray-400" size={20} />
            <input
              type="text"
              placeholder="Search orders..."
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
              className="w-full pl-10 pr-4 py-2 border rounded-lg"
            />
          </div>
        </div>

        <div className="bg-white rounded-lg border overflow-hidden">
          <table className="w-full">
            <thead className="bg-gray-50">
              <tr>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Order ID</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Customer</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Product</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Quantity</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Date</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Status</th>
                <th className="px-6 py-3 text-left text-sm font-semibold text-gray-900">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y">
              {filtered.map(order => {
                const customer = customers.find(c => c.id === parseInt(order.customerId));
                const product = products.find(p => p.id === parseInt(order.productId));
                return (
                  <tr key={order.id} className="hover:bg-gray-50">
                    <td className="px-6 py-4 text-sm text-gray-900">#{order.id}</td>
                    <td className="px-6 py-4 text-sm text-gray-900">{customer?.name || 'N/A'}</td>
                    <td className="px-6 py-4 text-sm text-gray-900">{product?.name || 'N/A'}</td>
                    <td className="px-6 py-4 text-sm text-gray-900">{order.quantity}</td>
                    <td className="px-6 py-4 text-sm text-gray-600">{order.date}</td>
                    <td className="px-6 py-4 text-sm">
                      <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                        order.status === 'completed' ? 'bg-green-100 text-green-800' :
                        order.status === 'pending' ? 'bg-yellow-100 text-yellow-800' :
                        'bg-blue-100 text-blue-800'
                      }`}>
                        {order.status}
                      </span>
                    </td>
                    <td className="px-6 py-4 text-sm">
                      <button onClick={() => openModal('order', order)} className="text-blue-600 hover:text-blue-800 mr-3">
                        <Edit2 size={18} />
                      </button>
                      <button onClick={() => handleDeleteOrder(order.id)} className="text-red-600 hover:text-red-800">
                        <Trash2 size={18} />
                      </button>
                    </td>
                  </tr>
                );
              })}
            </tbody>
          </table>
        </div>
      </div>
    );
  };

  const Modal = () => {
    if (!showModal) return null;

    return (
      <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
        <div className="bg-white rounded-lg p-6 w-full max-w-md">
          <h3 className="text-xl font-bold mb-4">
            {editingItem ? 'Edit' : 'Add'} {modalType.charAt(0).toUpperCase() + modalType.slice(1)}
          </h3>
          <form onSubmit={handleSubmit}>
            {modalType === 'product' && (
              <>
                <input name="name" defaultValue={editingItem?.name} placeholder="Product Name" className="w-full mb-3 px-3 py-2 border rounded" required />
                <input name="sku" defaultValue={editingItem?.sku} placeholder="SKU" className="w-full mb-3 px-3 py-2 border rounded" required />
                <input name="price" type="number" step="0.01" defaultValue={editingItem?.price} placeholder="Price" className="w-full mb-3 px-3 py-2 border rounded" required />
                <input name="stock" type="number" defaultValue={editingItem?.stock} placeholder="Stock" className="w-full mb-3 px-3 py-2 border rounded" required />
              </>
            )}
            {modalType === 'customer' && (
              <>
                <input name="name" defaultValue={editingItem?.name} placeholder="Customer Name" className="w-full mb-3 px-3 py-2 border rounded" required />
                <input name="email" type="email" defaultValue={editingItem?.email} placeholder="Email" className="w-full mb-3 px-3 py-2 border rounded" required />
                <input name="phone" defaultValue={editingItem?.phone} placeholder="Phone" className="w-full mb-3 px-3 py-2 border rounded" required />
                <input name="company" defaultValue={editingItem?.company} placeholder="Company" className="w-full mb-3 px-3 py-2 border rounded" />
              </>
            )}
            {modalType === 'order' && (
              <>
                <select name="customerId" defaultValue={editingItem?.customerId} className="w-full mb-3 px-3 py-2 border rounded" required>
                  <option value="">Select Customer</option>
                  {customers.map(c => <option key={c.id} value={c.id}>{c.name}</option>)}
                </select>
                <select name="productId" defaultValue={editingItem?.productId} className="w-full mb-3 px-3 py-2 border rounded" required>
                  <option value="">Select Product</option>
                  {products.map(p => <option key={p.id} value={p.id}>{p.name}</option>)}
                </select>
                <input name="quantity" type="number" defaultValue={editingItem?.quantity} placeholder="Quantity" className="w-full mb-3 px-3 py-2 border rounded" required />
                <select name="status" defaultValue={editingItem?.status || 'pending'} className="w-full mb-3 px-3 py-2 border rounded" required>
                  <option value="pending">Pending</option>
                  <option value="processing">Processing</option>
                  <option value="completed">Completed</option>
                </select>
              </>
            )}
            <div className="flex gap-2">
              <button type="submit" className="flex-1 bg-blue-600 text-white py-2 rounded hover:bg-blue-700">
                {editingItem ? 'Update' : 'Create'}
              </button>
              <button type="button" onClick={closeModal} className="flex-1 bg-gray-300 text-gray-700 py-2 rounded hover:bg-gray-400">
                Cancel
              </button>
            </div>
          </form>
        </div>
      </div>
    );
  };

  return (
    <div className="flex h-screen bg-gray-100">
      <div className="w-64 bg-gray-900 text-white">
        <div className="p-6">
          <h1 className="text-2xl font-bold">ERP System</h1>
        </div>
        <nav className="mt-6">
          <button
            onClick={() => { setActiveMenu('dashboard'); setSearchTerm(''); }}
            className={`w-full flex items-center gap-3 px-6 py-3 hover:bg-gray-800 ${activeMenu === 'dashboard' ? 'bg-gray-800 border-l-4 border-blue-500' : ''}`}
          >
            <LayoutDashboard size={20} />
            <span>Dashboard</span>
          </button>
          <button
            onClick={() => { setActiveMenu('products'); setSearchTerm(''); }}
            className={`w-full flex items-center gap-3 px-6 py-3 hover:bg-gray-800 ${activeMenu === 'products' ? 'bg-gray-800 border-l-4 border-blue-500' : ''}`}
          >
            <Package size={20} />
            <span>Products</span>
          </button>
          <button
            onClick={() => { setActiveMenu('customers'); setSearchTerm(''); }}
            className={`w-full flex items-center gap-3 px-6 py-3 hover:bg-gray-800 ${activeMenu === 'customers' ? 'bg-gray-800 border-l-4 border-blue-500' : ''}`}
          >
            <Users size={20} />
            <span>Customers</span>
          </button>
          <button
            onClick={() => { setActiveMenu('orders'); setSearchTerm(''); }}
            className={`w-full flex items-center gap-3 px-6 py-3 hover:bg-gray-800 ${activeMenu === 'orders' ? 'bg-gray-800 border-l-4 border-blue-500' : ''}`}
          >
            <ShoppingCart size={20} />
            <span>Orders</span>
          </button>
        </nav>
      </div>

      <div className="flex-1 overflow-auto">
        {activeMenu === 'dashboard' && <Dashboard />}
        {activeMenu === 'products' && <ProductsView />}
        {activeMenu === 'customers' && <CustomersView />}
        {activeMenu === 'orders' && <OrdersView />}
      </div>

      <Modal />
    </div>
  );
}
