# findAny/findFirst: Find primary address

- Only use `findFirst` if necessary.
- In most cases, `findAny` will be good enough.
- Example where `findFirst` makes sense, is when used with `sorted`.

## Basic

```java
List<Address> primaryAddresses = customerAccount.getAddresses().stream()
    .filter(Address::isPrimary)
    .collect(Collectors.toList());
Address primaryAddress = primaryAddresses.isEmpty() ? null : primaryAddresses.get(0);
```

## Advanced

```java
Address primaryAddress = customerAccount.getAddresses().stream()
    .filter(Address::isPrimary)
    .findAny()
    .orElse(null);
```

# anyMatch/noneMatch/allMatch: Check if any items are out of stock

## Basic

### Variant 1

```java
List<Item> outOfStockItems = items.stream()
    .filter(item -> item.getStock() == 0)
    .collect(Collectors.toList());
boolean anyOutOfStock = !outOfStockItems.isEmpty();
```

### Variant 2

```java
boolean anyOutOfStock = order.getItems().stream()
    .filter(item -> item.getStock() == 0)
    .count() > 0;
```

## Advanced

```java
boolean anyOutOfStock = order.getItems().stream()
    .anyMatch(item -> item.getStock() == 0);
```

# flatMap: Check if user has required permission

## Basic

```java
for (Role role : user.getRoles()) {
    for (String perm : role.getPermissions()) {
        if (perm.equals(requiredPermission)) {
            return; // has permission
        }
    }
}
throw new AccessDeniedException();
```

## Advanced

```java
user.getRoles().stream()
    .flatMap(role -> role.getPermissions().stream())
    .filter(perm -> perm.equals(requiredPermission))
    .findAny()
    .orElseThrow(() -> new AccessDeniedException());
```

# Removing empty Optionals from a List

## Basic

### Abstract example (with Optional anti-pattern)

```java
List<String> results = new ArrayList<>();
for (Optional<String> opt : optionals) {
    if (opt.isPresent()) {
         results.add(opt.get());
    }
}
```

### Abstract example (without Optional anti-pattern)

```java
List<String> results = new ArrayList<>();
for (Optional<String> opt : optionals) {
    opt.ifPresent(result -> results.add(result));
}
```

## Advanced

```java
List<String> results = optionals.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());
```

# Collectors.joining: Product categories separated by a comma and a space

## Basic

### Without streams (before Java 8)

```java
String delimiter = ", ";
StringBuilder sb = new StringBuilder();
boolean isFirst = true;
for (Product product : products) {
    String category = product.getCategories();
    if (!isFirst) {
        sb.append(delimiter);
    } else {
        isFirst = false;
    }
    sb.append(category);
}
String result = sb.toString();
```

### With streams

```java
List<String> names = products.stream()
    .map(product -> product.getCategories())
    .collect(Collectors.toList());
String result = String.join(", ", names); // since Java 8
```

## Advanced

```java
String result = products.stream()
    .map(Product::getCategories)
    .collect(Collectors.joining(", "));
```

# min/max: Newest order

## Basic

```java
Optional<Order> newestOrder = orders.stream()
    .sorted(Comparator.comparing(Order::getCreationDate).reversed())
    .findFirst();
```

## Advanced

```java
Optional<Order> newestOrder = orders.stream()
    .max(Comparator.comparing(Order::getCreationDate));
```

# mapToInt/mapToLong/mapToDouble/mapToObj: Creating 5 order objects with the index in the name in a list

## Basic

```java
List<Order> ordersList = new ArrayList<>();
for (int i = 1; i <= 5; i++) {
    ordersList.add(new Order("Order #" + i));
}
```

## Advanced

```java
List<Order> ordersList = IntStream.rangeClosed(1, 5)
    .mapToObj(i -> new Order("Order #" + i))
    .collect(Collectors.toList());
```

# reduce: Sum of order totals

## Basic

```java
BigDecimal total = BigDecimal.ZERO;
for (Order order : orders) {
    total = total.add(order.getTotalAmount());
}
```

## Advanced

```java
BigDecimal total = orders.stream()
    .map(Order::getTotalAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

# sum: Sum of all radii of circles

## Basic

```java
double totalRadius = shapes.stream()
    .filter(s -> s instanceof Circle)
    .map(s -> (Circle) s)
    .map(Circle::getRadius)
    .reduce(0.0, Double::sum);
```

## Advanced

```java
double totalRadius = shapes.stream()
    .filter(Circle.class::isInstance)
    .map(Circle.class::cast)
    .mapToDouble(Circle::getRadius)
    .sum();
```

# parallelStream

- Only change: Replace `.stream` with `.parallelStream`
- Be careful about performance overhead!

## Example

```java
public class ParallelStreamDemo {

    public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 100_000)
                .boxed()
                .toList();

        long result = numbers.parallelStream()
                .mapToLong(ParallelStreamDemo::heavyComputation)
                .sum();
    }

    private static long heavyComputation(long number) {
        long result = 0;
        for (int i = 0; i < 1000; i++) {
            result += (long) Math.sqrt(number * i);
        }
        return result;
    }
}
```

# sorted: Favorite products in stock, ordered by product name

## Basic

```java
List<Product> favorites = new ArrayList<>();
for (Product product : user.getFavorites()) {
    if (product.isInStock()) {
        favorites.add(product);
    }
}
Collections.sort(favorites, Comparator.comparing(Product::getName));
```

## Advanced

```java
user.getFavorites().stream()
    .filter(Product::isInStock)
    .sorted(Comparator.comparing(Product::getName))
    .collect(Collectors.toList());
```

# limit: Top 3 rated products

## Basic

```java
List<Product> sortedProducts = homepageProducts.stream()
    .sorted(Comparator.comparing(Product::getRating).reversed())
    .collect(Collectors.toList());

List<Product> topThree = sortedProducts.size() > 3
    ? sortedProducts.subList(0, 3)
    : sortedProducts;
```

## Advanced

```java
List<Product> topThree = homepageProducts.stream()
    .sorted(Comparator.comparing(Product::getRating).reversed())
    .limit(3)
    .collect(Collectors.toList());
```

# count: Amount of unread messages

## Basic

```java
List<Message> unreadMessages = messageService.getMessages(customer)
    .stream()
    .filter(message -> !message.isRead())
    .collect(Collectors.toList());
int unread = unreadMessages.size();
```

## Advanced

```java
long unread = messageService.getMessages(customer)
    .stream()
    .filter(message -> !message.isRead())
    .count();
```

# distinct: Getting all discount codes used in orders alphabetically sorted

## Basic

```java
Set<String> discountCodesSet = new HashSet<>();
for (Order order : orders) {
    discountCodesSet.add(order.getDiscountCode());
}
List<String> discountCodesList = new ArrayList<>(discountCodesSet);
Collections.sort(discountCodesList);
```

## Advanced

```java
orders.stream()
     .map(Order::getDiscountCode)
     .distinct()
     .sorted()
     .toList(); // since Java 16
```

# Collectors.toSet: Discount codes used in orders without duplicates

## Basic

```java
List<String> codes = orders.stream()
    .map(Order::getDiscountCode)
    .collect(Collectors.toList());
Set<String> uniqueCodes = new HashSet<>(codes);
```

## Advanced

```java
Set<String> uniqueCodes = orders.stream()
    .map(Order::getDiscountCode)
    .collect(Collectors.toSet());
```

# Collectors.toMap: Cheapest Product per Category

- Similar to `Collectors.groupingBy`, when keys are unique (every key is mapped to **exactly** one value).
- An optional third parameter can be provided, which allows to define how to handle cases where a key maps to multiple values.
- If no third parameter is provided, it will result in an exception if not unique.

## Basic

```java
Map<String, Product> cheapestByCategory = new HashMap<>();
for (Product product : products) {
    String category = product.getCategory();
    if (!cheapestByCategory.containsKey(category)) {
        cheapestByCategory.put(category, product);
    } else {
        Product currentCheapest = cheapestByCategory.get(category);
        if (product.getPrice().compareTo(currentCheapest.getPrice()) < 0) {
            cheapestByCategory.put(category, product);
        }
    }
}
```

## Advanced

```java
Map<String, Product> cheapestByCategory = products.stream()
    .collect(Collectors.toMap(
        Product::getCategory,
        Function.identity(),   // equivalent to: p -> p
        BinaryOperator.minBy(  // optional
            Comparator.comparing(Product::getPrice)
        )
    ));
```

# Collectors.groupingBy: List of orders grouped by customer

Similar to `Collectors.toMap`, when keys are mapped to multiple values.

## Basic

```java
Map<String, List<Order>> ordersByCustomer = new HashMap<>();
for (Order order : orders) {
    String customerId = order.getCustomerId();
    ordersByCustomer
        .computeIfAbsent(customerId, c -> new ArrayList<>())
        .add(order);
}
```

## Advanced

```java
Map<String, List<Order>> ordersByCustomer = orders.stream()
    .collect(Collectors.groupingBy(Order::getCustomerId));
```

# Collectors.mapping: Group product names by category

Most useful when used together with other Collectors, like `groupingBy` and `partitioningBy`.

## Basic

```java
Map<String, List<String>> productsByCategory = new HashMap<>();
for (Product product : products) {
    productsByCategory
        .computeIfAbsent(product.getCategory(), k -> new ArrayList<>())
        .add(product.getName());
}
```

## Advanced

```java
Map<String, List<String>> productsByCategory = products.stream()
    .collect(Collectors.groupingBy(
        Product::getCategory,
        Collectors.mapping(Product::getName, Collectors.toList())
    ));
```

# Collectors.counting: Count of times an item was sold by item name

## Basic

```java
Map<String, Long> mapNameToSales = new HashMap<>();
for (Order order : orders) {
    for (Item item : order.getItems()) {
        mapNameToSales.put(item.getName(), mapNameToSales.getOrDefault(item.getName(), 0) + 1);
    }
}
```

## Advanced

```java
Map<String, Long> mapNameToSales = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.groupingBy(Item::getName, Collectors.counting()));
```

# summingInt/averagingInt (also: `Double`/`Long`)

## Basic

```java
int totalQuantity = 0;
for (Order order : orders) {
    for (Item item : order.getItems()) {
        totalQuantity += item.getQuantity();
    }
}
```

## Advanced

```java
int totalQuantity = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.summingInt(Item::getQuantity));
```

# summarizingInt (also: `Double`/`Long`)

## Basic

```java
int sum = 0;
int count = 0;
int min = Integer.MAX_VALUE;
int max = Integer.MIN_VALUE;
for (Order order : orders) {
    for (Item item : order.getItems()) {
        int qty = item.getQuantity();
        sum += qty;
        count++;
        if (qty < min) {
            min = qty;
        }
        if (qty > max) {
            max = qty;
        }
    }
}
double average = count == 0 ? 0 : (double) sum / count;
```

## Advanced

```java
IntSummaryStatistics stats = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.summarizingInt(Item::getQuantity));

// Retrieve statistics:
// stats.getCount(), stats.getSum(), stats.getMin(), stats.getMax(), stats.getAverage()
```

# Collectors.partitioningBy: Split products up into available and out of stock

## Basic

```java
List<Product> availableProducts = new ArrayList<>();
List<Product> outOfStockProducts = new ArrayList<>();

for (Product product : products) {
    if (product.getStock() > 0) {
        availableProducts.add(product);
    } else {
        outOfStockProducts.add(product);
    }
}
```

## Advanced

```java
Map<Boolean, List<Product>> partitionedProducts = products.stream()
    .collect(Collectors.partitioningBy(product -> product.getStock() > 0));

List<Product> availableProducts = partitionedProducts.get(true);
List<Product> outOfStockProducts = partitionedProducts.get(false);
```

# Collectors.teeing: Cheapest and most expensive products

## Basic

```java
Product cheapest = products.get(0);
Product mostExpensive = products.get(0);
for (Product p : products) {
    if (p.getPrice().compareTo(cheapest.getPrice()) < 0) {
        cheapest = p;
    }
    if (p.getPrice().compareTo(mostExpensive.getPrice()) > 0) {
        mostExpensive = p;
    }
}
Pair<Product, Product> priceRange = new Pair<>(cheapest, mostExpensive);
```

## Advanced

```java
Pair<Product, Product> priceRange = products.stream()
    .collect(Collectors.teeing(
         Collectors.minBy(Comparator.comparing(Product::getPrice)),
         Collectors.maxBy(Comparator.comparing(Product::getPrice)),
         (minOpt, maxOpt) -> new Pair<>(minOpt.orElse(null), maxOpt.orElse(null))
    ));
```

# takeWhile/dropWhile: Orders placed before and during/after a sale (since Java 9)

Examples assume the data is already sorted.

## Basic

```java
List<Order> ordersBeforeSale = new ArrayList<>();
List<Order> ordersAfterSale = new ArrayList<>();
boolean beforeSale = true;
for (Order order : orders) {
    if (beforeSale && order.getOrderDate().isBefore(saleStart)) {
         ordersBeforeSale.add(order);
    } else {
         beforeSale = false;
         ordersAfterSale.add(order);
    }
}
```

## Advanced

```java
List<Order> ordersBeforeSale = orders.stream()
    .takeWhile(order -> order.getOrderDate().isBefore(saleStart))
    .collect(Collectors.toList());
List<Order> ordersAfterSale = orders.stream()
    .dropWhile(order -> order.getOrderDate().isBefore(saleStart))
    .collect(Collectors.toList());
```

# Combining comparators

## Basic

```java
Map<String, Integer> totalStockPerProduct = new HashMap<>();
for (ProductStock stock : stockList) {
    String productId = stock.getProductId();
    int currentTotal = totalStockPerProduct.getOrDefault(productId, 0);
    totalStockPerProduct.put(productId, currentTotal + stock.getQuantity());
}
```

## Advanced

```java
Map<String, Integer> totalStockPerProduct = stockList.stream()
     .collect(Collectors.groupingBy(
         ProductStock::getProductId,
         Collectors.summingInt(ProductStock::getQuantity)
     ));
```