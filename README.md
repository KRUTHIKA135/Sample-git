// === ITEM SERVICE ===

// Item.java (Entity)
package com.synechron.itemservice.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.NotBlank;

@Entity
public class Item {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Item name is required")
    private String name;

    @DecimalMin(value = "0.0", message = "Price must be greater than or equal to 0")
    private double price;

    // Getters and Setters
}

// ItemRepository.java
package com.synechron.itemservice.repository;

import com.synechron.itemservice.entity.Item;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ItemRepository extends JpaRepository<Item, Long> {}

// ItemService.java
package com.synechron.itemservice.service;

import com.synechron.itemservice.entity.Item;
import com.synechron.itemservice.repository.ItemRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ItemService {
    @Autowired
    private ItemRepository itemRepository;

    public List<Item> getAllItems() {
        return itemRepository.findAll();
    }

    public Item saveItem(Item item) {
        return itemRepository.save(item);
    }
}

// ItemController.java
package com.synechron.itemservice.controller;

import com.synechron.itemservice.entity.Item;
import com.synechron.itemservice.service.ItemService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/item")
public class ItemController {

    private static final String ITEM_SERVICE_URL = "http://localhost:8081/item";

    @Autowired
    private ItemService itemService;

    @GetMapping
    public List<Item> getAllItems() {
        return itemService.getAllItems();
    }

    @PostMapping
    public Item createItem(@RequestBody @Valid Item item) {
        return itemService.saveItem(item);
    }
}



// === ORDER SERVICE ===

// Order.java (Entity)
package com.synechron.orderservice.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;

import java.sql.Date;

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long orderId;

    @NotBlank(message = "Customer name is required")
    private String customerName;

    @NotBlank(message = "Item name is required")
    private String itemName;

    private int quantity;

    private Date orderDate;

    // Getters and Setters
}

// OrderRepository.java
package com.synechron.orderservice.repository;

import com.synechron.orderservice.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;

public interface OrderRepository extends JpaRepository<Order, Long> {}

// Feign Client to call ItemService
package com.synechron.orderservice.feign;

import com.synechron.orderservice.model.Item;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.List;

@FeignClient(name = "item-service")
public interface ItemClient {
    @GetMapping("/item")
    List<Item> getAllItems();
}

// Item.java (model in order service)
package com.synechron.orderservice.model;

public class Item {
    private Long id;
    private String name;
    private double price;

    // Getters and Setters
}

// OrderService.java
package com.synechron.orderservice.service;

import com.synechron.orderservice.entity.Order;
import com.synechron.orderservice.repository.OrderRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }

    public Order saveOrder(Order order) {
        return orderRepository.save(order);
    }
}

// OrderController.java
package com.synechron.orderservice.controller;

import com.synechron.orderservice.entity.Order;
import com.synechron.orderservice.feign.ItemClient;
import com.synechron.orderservice.model.Item;
import com.synechron.orderservice.service.OrderService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/order")
