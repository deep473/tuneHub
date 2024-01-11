package com.example.demo.controller;


import org.json.JSONObject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.example.demo.entities.Users;
import com.example.demo.services.UsersService;
import com.razorpay.Order;
import com.razorpay.RazorpayClient;
import com.razorpay.RazorpayException;

import jakarta.servlet.http.HttpSession;

@Controller
public class PaymentController {
	@Autowired
	UsersService service;

	@GetMapping("/pay")
	public String pay() {
		return "pay";
	}

	@SuppressWarnings("finally")
	@PostMapping("/createOrder")
	@ResponseBody
	public String createOrder(HttpSession session) {

		int  amount  = 5000;
		Order order=null;
		try {
			RazorpayClient razorpay=new RazorpayClient("rzp_test_pvGIc9JvXWZTVS", "UlnLCUb8lRxvBKRyIYRYWrEO");

			JSONObject orderRequest = new JSONObject();
			orderRequest.put("amount", amount*100); // amount in the smallest currency unit
			orderRequest.put("currency", "INR");
			orderRequest.put("receipt", "order_rcptid_11");

			order = razorpay.orders.create(orderRequest);

			String mail =  (String) session.getAttribute("email");

			Users u = service.getUser(mail);
			u.setPremium(true);
			service.updateUser(u);

		} catch (RazorpayException e) {
			e.printStackTrace();
		}
		finally {
			return order.toString();
		}
	}	
}



<!DOCTYPE html>
<html xmlns:th="https://www.thymeleaf.org">
<head>
<meta charset="ISO-8859-1">
	
	<title>View Course</title>
	<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
	<script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>

</head>
<body>
<div>

		<h1>Why premium ? </h1>
		<p>text...................</p>
		
    	<form id="payment-form">
	        <button type="submit" class="buy-button" >BUY</button>
	    </form>
   
</div>

<script>
$(document).ready(function() {
    $(".buy-button").click(function(e) {
        e.preventDefault();
        var form = $(this).closest('form');
        
        
        createOrder();
    });
});

function createOrder() {
	
    $.post("/createOrder")
        .done(function(order) {
            order = JSON.parse(order);
            var options = {
                "key": "rzp_test_pvGIc9JvXWZTVS",
                "amount": order.amount_due.toString(),
                "currency": "INR",
                "name": "Tune Hub",
                "description": "Test Transaction",
                "order_id": order.id,
                "handler": function (response) {
                    verifyPayment(response.razorpay_order_id, response.razorpay_payment_id, response.razorpay_signature);
                },
                "prefill": {
                    "name": "Your Name",
                    "email": "test@example.com",
                    "contact": "9999999999"
                },
                "notes": {
                    "address": "Your Address"
                },
                "theme": {
                    "color": "#F37254"
                }
            };
            var rzp1 = new Razorpay(options);
            rzp1.open();
        })
        .fail(function(error) {
            console.error("Error:", error);
        });
}

function verifyPayment(orderId, paymentId, signature) {
     $.post("/verify", { orderId: orderId, paymentId: paymentId, signature: signature })
         .done(function(isValid) {
             if (isValid) {
                 console.log("Payment successful");
             } else {
                 console.log("Payment failed");
             }
         })
         .fail(function(error) {
             console.error("Error:", error);
         });
}
</script>
</body>
</html>
