import 'package:flutter/material.dart';

void main() {
  runApp(const EverythingByBettyApp());
}

class EverythingByBettyApp extends StatelessWidget {
  const EverythingByBettyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Everything by Betty Manager',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        brightness: Brightness.dark,
        primarySwatch: Colors.amber,
        scaffoldBackgroundColor: const Color(0xFF121212),
        cardColor: const Color(0xFF1E1E1E),
        colorScheme: const ColorScheme.dark(
          primary: Color(0xFFFF9800), // Culinary Amber
          secondary: Color(0xFFFF5722), // Deep Orange Accent
          surface: Color(0xFF1E1E1E),
        ),
        useMaterial3: true,
      ),
      home: const MainNavigationScreen(),
    );
  }
}

// ==========================================
// DATA MODELS & HELPERS
// ==========================================
class Product {
  final String id;
  final String name;
  final String category;
  int stockQuantity;
  final double costPrice;
  final double sellingPrice;

  Product({
    required this.id,
    required this.name,
    required this.category,
    required this.stockQuantity,
    required this.costPrice,
    required this.sellingPrice,
  });
}

class Sale {
  final String id;
  final DateTime timestamp;
  final String productId;
  final String productName;
  final int quantitySold;
  final double totalPrice;
  final String paymentMethod;

  Sale({
    required this.id,
    required this.timestamp,
    required this.productId,
    required this.productName,
    required this.quantitySold,
    required this.totalPrice,
    required this.paymentMethod,
  });
}

class Expense {
  final String id;
  final DateTime date;
  final String category;
  final double amount;
  final String description;

  Expense({
    required this.id,
    required this.date,
    required this.category,
    required this.amount,
    required this.description,
  });
}

// Custom Helper Class for the Feed
class Tuple<T1, T2, T3, T4> {
  final T1 item1;
  final T2 item2;
  final T3 item3;
  final T4 item4;

  Tuple(this.item1, this.item2, this.item3, this.item4);
}

// ==========================================
// MOCK DATA STORE (Simulating Database State)
// ==========================================
final ValueNotifier<List<Product>> productsNotifier = ValueNotifier([
  Product(id: '1', name: 'Jollof Rice (Standard Pack)', category: 'Meals', stockQuantity: 25, costPrice: 1500, sellingPrice: 2500),
  Product(id: '2', name: 'Gourmet Meat Pies (Box of 6)', category: 'Pastries', stockQuantity: 4, costPrice: 3000, sellingPrice: 5000),
  Product(id: '3', name: 'Fried Rice Combo', category: 'Meals', stockQuantity: 15, costPrice: 1800, sellingPrice: 3000),
  Product(id: '4', name: 'Signature Spices', category: 'Retail', stockQuantity: 30, costPrice: 800, sellingPrice: 1500),
]);

final ValueNotifier<List<Sale>> salesNotifier = ValueNotifier([
  Sale(id: 's1', timestamp: DateTime.now(), productId: '1', productName: 'Jollof Rice (Standard Pack)', quantitySold: 2, totalPrice: 5000, paymentMethod: 'Bank Transfer'),
]);

final ValueNotifier<List<Expense>> expensesNotifier = ValueNotifier([
  Expense(id: 'e1', date: DateTime.now(), category: 'Ingredients', amount: 4500, description: 'Purchased chicken, spices, and vegetable oil'),
]);

// ==========================================
// NAVIGATION WRAPPER
// ==========================================
class MainNavigationScreen extends StatefulWidget {
  const MainNavigationScreen({super.key});

  @override
  State<MainNavigationScreen> createState() => _MainNavigationScreenState();
}

class _MainNavigationScreenState extends State<MainNavigationScreen> {
  int _selectedIndex = 0;

  final List<Widget> _screens = [
    const DashboardScreen(),
    const InventoryScreen(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: IndexedStack(index: _selectedIndex, children: _screens),
      bottomNavigationBar: NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (index) => setState(() => _selectedIndex = index),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.dashboard_outlined), selectedIcon: Icon(Icons.dashboard), label: 'Dashboard'),
          NavigationDestination(icon: Icon(Icons.inventory_2_outlined), selectedIcon: Icon(Icons.inventory_2), label: 'Inventory'),
        ],
      ),
    );
  }
}

// ==========================================
// 1. DASHBOARD SCREEN
// ==========================================
class DashboardScreen extends StatelessWidget {
  const DashboardScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Everything by Betty', style: TextStyle(fontWeight: FontWeight.bold)),
        centerTitle: false,
      ),
      body: AnimatedBuilder(
        animation: Listenable.merge([salesNotifier, expensesNotifier, productsNotifier]),
        builder: (context, _) {
          double totalSales = salesNotifier.value.fold(0.0, (sum, item) => sum + item.totalPrice);
          double totalExpenses = expensesNotifier.value.fold(0.0, (sum, item) => sum + item.amount);
          double netProfit = totalSales - totalExpenses;

          List<Product> lowStockItems = productsNotifier.value.where((p) => p.stockQuantity <= 5).toList();

          return SingleChildScrollView(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Financial KPI Metrics
                Row(
                  children: [
                    Expanded(child: _buildStatCard('Sales', '₦${totalSales.toStringAsFixed(0)}', Colors.greenAccent)),
                    const SizedBox(width: 12),
                    Expanded(child: _buildStatCard('Expenses', '₦${totalExpenses.toStringAsFixed(0)}', Colors.redAccent)),
                  ],
                ),
                _buildStatCard(
                  'Net Profit Today', 
                  '₦${netProfit.toStringAsFixed(0)}', 
                  netProfit >= 0 ? Colors.amberAccent : Colors.redAccent,
                  isFullWidth: true
                ),
                const SizedBox(height: 24),

                // Low Stock Alerts section
                if (lowStockItems.isNotEmpty) ...[
                  const Text('🚨 Low Stock Alerts', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.orangeAccent)),
                  const SizedBox(height: 8),
                  Card(
                    color: Colors.amber.withOpacity(0.1),
                    shape: RoundedRectangleBorder(side: const BorderSide(color: Colors.amber, width: 0.5), borderRadius: BorderRadius.circular(12)),
                    child: Column(
                      children: lowStockItems.map((product) => ListTile(
                        leading: const Icon(Icons.warning_amber_rounded, color: Colors.amber),
                        title: Text(product.name, style: const TextStyle(fontWeight: FontWeight.w600)),
                        trailing: Text('${product.stockQuantity} remaining', style: const TextStyle(color: Colors.redAccent, fontWeight: FontWeight.bold)),
                      )).toList(),
                    ),
                  ),
                  const SizedBox(height: 24),
                ],

                // Action Buttons Matrix
                const Text('Quick Actions', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                const SizedBox(height: 12),
                Row(
                  children: [
                    Expanded(
                      child: ElevatedButton.icon(
                        onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => const LogSaleScreen())),
                        icon: const Icon(Icons.point_of_sale),
                        label: const Text('Log a Sale'),
                        style: ElevatedButton.styleFrom(
                          backgroundColor: Theme.of(context).colorScheme.primary,
                          foregroundColor: Colors.black,
                          padding: const EdgeInsets.symmetric(vertical: 16),
                          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                        ),
                      ),
                    ),
                    const SizedBox(width: 12),
                    Expanded(
                      child: ElevatedButton.icon(
                        onPressed: () => Navigator.push(context, MaterialPageRoute(builder: (_) => const AddExpenseScreen())),
                        icon: const Icon(Icons.money_off),
                        label: const Text('Add Expense'),
                        style: ElevatedButton.styleFrom(
                          backgroundColor: Colors.grey[850],
                          foregroundColor: Colors.white,
                          padding: const EdgeInsets.symmetric(vertical: 16),
                          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                        ),
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 28),

                // Recent Activity Roll
                const Text('Recent Activity Log', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                const SizedBox(height: 12),
                _buildActivityList(),
              ],
            ),
          );
        },
      ),
    );
  }

  Widget _buildStatCard(String title, String money, Color color, {bool isFullWidth = false}) {
    return Container(
      width: isFullWidth ? double.infinity : null,
      margin: isFullWidth ? const EdgeInsets.only(top: 12) : EdgeInsets.zero,
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: const Color(0xFF1E1E1E),
        borderRadius: BorderRadius.circular(16),
        border: Border.all(color: Colors.white10),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(title, style: const TextStyle(color: Colors.grey, fontSize: 14)),
          const SizedBox(height: 8),
          Text(money, style: TextStyle(color: color, fontSize: 24, fontWeight: FontWeight.bold)),
        ],
      ),
    );
  }

  Widget _buildActivityList() {
    final activities = [
      ...salesNotifier.value.map((s) => Tuple(s.timestamp, '💰 Sold ${s.quantitySold}x ${s.productName}', '+₦${s.totalPrice.toStringAsFixed(0)}', Colors.greenAccent)),
      ...expensesNotifier.value.map((e) => Tuple(e.date, '📉 ${e.category}: ${e.description}', '-₦${e.amount.toStringAsFixed(0)}', Colors.redAccent)),
    ];
    activities.sort((a, b) => b.item1.compareTo(a.item1));

    if (activities.isEmpty) {
      return const Center(child: Padding(padding: EdgeInsets.all(16.0), child: Text('No activities recorded today.', style: TextStyle(color: Colors.grey))));
    }

    return ListView.separated(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      itemCount: activities.length,
      separatorBuilder: (_, __) => const Divider(color: Colors.white10),
      itemBuilder: (context, index) {
        final act = activities[index];
        return ListTile(
          contentPadding: EdgeInsets.zero,
          title: Text(act.item2, style: const TextStyle(fontSize: 14, fontWeight: FontWeight.w500)),
          subtitle: Text('${act.item1.hour.toString().padLeft(2, '0')}:${act.item1.minute.toString().padLeft(2, '0')}', style: const TextStyle(color: Colors.grey)),
          trailing: Text(act.item3, style: TextStyle(color: act.item4, fontWeight: FontWeight.bold, fontSize: 15)),
        );
      },
    );
  }
}

// ==========================================
// 2. LOG A SALE SCREEN
// ==========================================
class LogSaleScreen extends StatefulWidget {
  const LogSaleScreen({super.key});

  @override
  State<LogSaleScreen> createState() => _LogSaleScreenState();
}

class _LogSaleScreenState extends State<LogSaleScreen> {
  Product? _selectedProduct;
  final TextEditingController _qtyController = TextEditingController(text: '1');
  String _payMethod = 'Bank Transfer';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Log Fresh Sale')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('Select Product', style: TextStyle(fontWeight: FontWeight.w500)),
            const SizedBox(height: 8),
            DropdownButtonFormField<Product>(
              decoration: InputDecoration(filled: true, fillColor: const Color(0xFF1E1E1E), border: OutlineInputBorder(borderRadius: BorderRadius.circular(12))),
              value: _selectedProduct,
              hint: const Text('Choose product'),
              items: productsNotifier.value.map((p) => DropdownMenuItem(value: p, child: Text('${p.name} (Stock: ${p.stockQuantity})'))).toList(),
              onChanged: (val) => setState(() => _selectedProduct = val),
            ),
            const SizedBox(height: 20),
            const Text('Quantity Sold', style: TextStyle(fontWeight: FontWeight.w500)),
            const SizedBox(height: 8),
            TextField(
              controller: _qtyController,
              keyboardType: TextInputType.number,
              decoration: InputDecoration(filled: true, fillColor: const Color(0xFF1E1E1E), border: OutlineInputBorder(borderRadius: BorderRadius.circular(12))),
            ),
            const SizedBox(height: 20),
            const Text('Payment Mode', style: TextStyle(fontWeight: FontWeight.w500)),
            const SizedBox(height: 8),
            Row(
              children: ['Bank Transfer', 'Cash'].map((method) => Expanded(
                child: RadioListTile<String>(
                  title: Text(method),
                  value: method,
                  groupValue: _payMethod,
                  onChanged: (val) => setState(() => _payMethod = val!),
                ),
              )).toList(),
            ),
            const SizedBox(height: 40),
            SizedBox(
              width: double.infinity,
              height: 52,
              child: ElevatedButton(
                style: ElevatedButton.styleFrom(backgroundColor: Colors.amber, foregroundColor: Colors.black, shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12))),
                onPressed: () {
                  if (_selectedProduct == null) return;
                  int qty = int.tryParse(_qtyController.text) ?? 0;
                  if (qty <= 0 || qty > _selectedProduct!.stockQuantity) {
                    ScaffoldMessenger.of(context).showSnackBar(const SnackBar(content: Text('Invalid quantity or insufficient stock!')));
                    return;
                  }

                  // Deduct transactional quantities directly from application memory
                  _selectedProduct!.stockQuantity -= qty;
                  salesNotifier.value = [...salesNotifier.value, Sale(
                    id: DateTime.now().toString(),
                    timestamp: DateTime.now(),
                    productId: _selectedProduct!.id,
                    productName: _selectedProduct!.name,
                    quantitySold: qty,
                    totalPrice: _selectedProduct!.sellingPrice * qty,
                    paymentMethod: _payMethod,
                  )];

                  Navigator.pop(context);
                },
                child: const Text('Confirm Sale Transaction', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
              ),
            )
          ],
        ),
      ),
    );
  }
}

// ==========================================
// 3. ADD EXPENSE SCREEN
// ==========================================
class AddExpenseScreen extends StatefulWidget {
  const AddExpenseScreen({super.key});

  @override
  State<AddExpenseScreen> createState() => _AddExpenseScreenState();
}

class _AddExpenseScreenState extends State<AddExpenseScreen> {
  final TextEditingController _amountController = TextEditingController();
  final TextEditingController _descController = TextEditingController();
  String _category = 'Ingredients';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Record Expense')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(24.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text('Expense Category', style: TextStyle(fontWeight: FontWeight.w500)),
            const SizedBox(height: 8),
            DropdownButtonFormField<String>(
              decoration: InputDecoration(filled: true, fillColor: const Color(0xFF1E1E1E), border: OutlineInputBorder(borderRadius: BorderRadius.circular(12))),
              value: _category,
              items: ['Ingredients', 'Logistics', 'Packaging', 'Utilities'].map((cat) => DropdownMenuItem(value: cat, child: Text(cat))).toList(),
              onChanged: (val) => setState(() => _category = val!),
            ),
            const SizedBox(height: 20),
            const Text('Amount Spent (₦)', style: TextStyle(fontWeight: FontWeight.w500)),
            const SizedBox(height: 8),
            TextField(
              controller: _amountController,
              keyboardType: TextInputType.number,
              decoration: InputDecoration(filled: true, fillColor: const Color(0xFF1E1E1E), border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)), hintText: '0.00'),
            ),
            const SizedBox(height: 20),
            const Text('Expense Notes / Description', style: TextStyle(fontWeight: FontWeight.w500)),
            const SizedBox(height: 8),
            TextField(
              controller: _descController,
              maxLines: 3,
              decoration: InputDecoration(filled: true, fillColor: const Color(0xFF1E1E1E), border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)), hintText: 'What was this cash used for?'),
            ),
            const SizedBox(height: 40),
            SizedBox(
              width: double.infinity,
              height: 52,
              child: ElevatedButton(
                style: ElevatedButton.styleFrom(backgroundColor: Colors.grey[200], foregroundColor: Colors.black, shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12))),
                onPressed: () {
                  double amount = double.tryParse(_amountController.text) ?? 0.0;
                  if (amount <= 0 || _descController.text.isEmpty) return;

                  expensesNotifier.value = [...expensesNotifier.value, Expense(
                    id: DateTime.now().toString(),
                    date: DateTime.now(),
                    category: _category,
                    amount: amount,
                    description: _descController.text,
                  )];

                  Navigator.pop(context);
                },
                child: const Text('Log Expense Outflow', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
              ),
            )
          ],
        ),
      ),
    );
  }
}

// ==========================================
// 4. INVENTORY SHEET SCREEN (COMPLETED)
// ==========================================
class InventoryScreen extends StatelessWidget {
  const InventoryScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Inventory Master Ledger')),
      body: ValueListenableBuilder<List<Product>>(
        valueListenable: productsNotifier,
        builder: (context, products, _) {
          if (products.isEmpty) {
            return const Center(child: Text('No products available in inventory.'));
          }

          return ListView.builder(
            padding: const EdgeInsets.all(16),
            itemCount: products.length,
            itemBuilder: (context, index) {
              final product = products[index];
              final bool isLowStock = product.stockQuantity <= 5;

              return Card(
                margin: const EdgeInsets.only(bottom: 12),
                child: Padding(
                  padding: const EdgeInsets.all(16.0),
                  child: Row(
                    children: [
                      Expanded(
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(
                              product.name,
                              style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold),
                            ),
                            const SizedBox(height: 4),
                            Text(
                              'Category: ${product.category}',
                              style: TextStyle(color: Colors.grey[400], fontSize: 13),
                            ),
                            const SizedBox(height: 8),
                            Row(
                              children: [
                                Text('Cost: ₦${product.costPrice.toStringAsFixed(0)}', style: const TextStyle(fontSize: 12, color: Colors.grey)),
                                const SizedBox(width: 16),
                                Text('Sell: ₦${product.sellingPrice.toStringAsFixed(0)}', style: const TextStyle(fontSize: 12, color: Colors.amber)),
                              ],
                            ),
                          ],
                        ),
                      ),
                      Column(
                        crossAxisAlignment: CrossAxisAlignment.end,
                        children: [
                          const Text('Stock Level', style: TextStyle(fontSize: 12, color: Colors.grey)),
                          const SizedBox(height: 4),
                          Container(
                            padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
                            decoration: BoxDecoration(
                              color: isLowStock ? Colors.red.withOpacity(0.2) : Colors.green.withOpacity(0.2),
                              borderRadius: BorderRadius.circular(8),
                              border: Border.all(
                                color: isLowStock ? Colors.redAccent : Colors.greenAccent,
                                width: 1,
                              ),
                            ),
                            child: Text(
                              '${product.stockQuantity} pcs',
                              style: TextStyle(
                                color: isLowStock ? Colors.redAccent : Colors.greenAccent,
                                fontWeight: FontWeight.bold,
                              ),
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
