/bookstore
  /index.html
  /styles.css
  /script.js
  /images
    - book1.jpg
    - book2.jpg
  /assets
    - logo.png
    <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Bookstore</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header>
    <div class="logo">
      <img src="assets/logo.png" alt="My Bookstore Logo">
    </div>
    <nav>
      <ul>
        <li><a href="#books">Books</a></li>
        <li><a href="#about">About</a></li>
        <li><a href="#contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <section id="books">
      <h1>Our Books</h1>
      <div class="book-container">
        <div class="book-card">
          <img src="images/book1.jpg" alt="Book 1">
          <h3>Book Title 1</h3>
          <p>$19.99</p>
          <button class="buy-btn" onclick="buyBook('book1')">Buy Now</button>
        </div>
        <div class="book-card">
          <img src="images/book2.jpg" alt="Book 2">
          <h3>Book Title 2</h3>
          <p>$14.99</p>
          <button class="buy-btn" onclick="buyBook('book2')">Buy Now</button>
        </div>
      </div>
    </section>

    <section id="about">
      <h2>About Us</h2>
      <p>Welcome to our bookstore! We offer a wide variety of books that will inspire and entertain you. Explore our collection!</p>
    </section>

    <section id="contact">
      <h2>Contact Us</h2>
      <p>Email: contact@mybookstore.com</p>
    </section>
  </main>

  <footer>
    <p>&copy; 2025 My Bookstore. All Rights Reserved.</p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

header {
  background-color: #333;
  color: white;
  padding: 10px;
  text-align: center;
}

header nav ul {
  list-style-type: none;
  padding: 0;
}

header nav ul li {
  display: inline;
  margin: 0 20px;
}

header nav ul li a {
  color: white;
  text-decoration: none;
}

#books {
  text-align: center;
  padding: 50px 20px;
}

.book-container {
  display: flex;
  justify-content: center;
  gap: 30px;
}

.book-card {
  width: 200px;
  text-align: center;
  padding: 10px;
  border: 1px solid #ddd;
}

.book-card img {
  width: 100%;
}

.buy-btn {
  background-color: #28a745;
  color: white;
  padding: 10px;
  border: none;
  cursor: pointer;
  margin-top: 10px;
}

.buy-btn:hover {
  background-color: #218838;
}

footer {
  background-color: #333;
  color: white;
  text-align: center;
  padding: 10px;
}

footer p {
  margin: 0;
}
function buyBook(book) {
  alert(`You selected ${book}. Redirecting to payment...`);
  // Here, you'd add the code to process the payment
  // For example, you can integrate with Stripe or PayPal APIs.
}
<script src="https://js.stripe.com/v3/"></script>
const stripe = Stripe('your-publishable-key-here'); // Your Stripe Public Key

function buyBook(book) {
  fetch('/create-checkout-session', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ book: book })
  })
  .then(response => response.json())
  .then(sessionId => {
    stripe.redirectToCheckout({ sessionId: sessionId });
  })
  .catch(error => console.error('Error:', error));
}
npm init -y
npm install express stripe
const express = require('express');
const stripe = require('stripe')('your-secret-key-here'); // Your Stripe Secret Key
const app = express();

app.use(express.static('public')); // Serve static files (HTML, CSS, JS)
app.use(express.json()); // Parse JSON bodies

// Route to create a Stripe Checkout session
app.post('/create-checkout-session', async (req, res) => {
  const { book } = req.body;

  // Set the price based on the selected book
  const prices = {
    book1: 1999, // $19.99
    book2: 1499, // $14.99
  };

  try {
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      line_items: [
        {
          price_data: {
            currency: 'usd',
            product_data: {
              name: book,
            },
            unit_amount: prices[book],
          },
          quantity: 1,
        },
      ],
      mode: 'payment',
      success_url: `${process.env.BASE_URL}/success.html`,
      cancel_url: `${process.env.BASE_URL}/cancel.html`,
    });

    res.json({ sessionId: session.id });
  } catch (error) {
    console.error('Error creating checkout session:', error);
    res.status(500).send('Internal Server Error');
  }
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
node server.js
