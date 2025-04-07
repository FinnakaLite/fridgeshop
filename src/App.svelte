<script>
  import { onMount } from 'svelte';

  let products = [];
  let loading = true;
  let error = null;

  // Base Vipps payment URL
  const VIPPS_BASE_URL = "https://qr.vipps.no/..."; // Replace with your actual Vipps payment URL
  
  // Base image path
  const IMAGE_BASE_PATH = "/img/"; 

  // Function to convert price to Vipps amount (multiply by 100 and round to nearest integer)
  function priceToVippsAmount(price) {
    return Math.round(price * 100);
  }

  // Function to generate Vipps payment link
  function generateVippsLink(price) {
    const amount = priceToVippsAmount(price);
    return `${VIPPS_BASE_URL}?amount=${amount}`;
  }
  
  // Function to generate image path
  function generateImagePath(productId) {
    return `${IMAGE_BASE_PATH}${productId}.jpg`;
  }

  async function loadProducts() {
    try {
      const response = await fetch('/data/products.json');
      if (!response.ok) throw new Error('Failed to load products');
      const data = await response.json();
      // Filter out disabled products and add Vipps links and image paths
      products = data.products
        .filter(product => !product.disabled)
        .map(product => ({
          ...product,
          link: generateVippsLink(product.price),
          image: generateImagePath(product.id)
        }));
    } catch (err) {
      console.error('Error loading products:', err);
      error = 'Failed to load products. Please try again later.';
    } finally {
      loading = false;
    }
  }

  onMount(loadProducts);

  function handleBuy(productId) {
    // Handle buy button click
    const product = products.find(p => p.id === productId);
    if (product && !product.soldOut && product.stock > 0) {
      window.location.href = product.link;
    }
  }
</script>

<main>
  <h1>Fridge Shop</h1>
  
  <div class="welcome-section">
    <h2>Welcome to Fridge Shop</h2>
    <p>Discover our selection of premium beverages. We offer a variety of drinks to suit every taste and occasion.</p>
  </div>
  
  {#if loading}
    <div class="loading">Loading products...</div>
  {:else if error}
    <div class="error">{error}</div>
  {:else}
    <div class="product-list">
      {#each products as product}
        <div class="product-card">
          <div class="product-image">
            <img src={product.image} alt={product.name} />
          </div>
          <div class="product-content">
            <div class="product-info">
              <h2>{product.name}</h2>
              <p class="description">{product.description}</p>
              <p class="stock">In Stock: {product.stock}</p>
            </div>
            <div class="product-action">
              <div class="price-action-container">
                <p class="price">{product.price} kr</p>
                {#if product.soldOut || product.stock === 0}
                  <button class="buy-button sold-out" disabled>Sold Out</button>
                {:else}
                  <button class="buy-button" on:click={() => handleBuy(product.id)}>Buy with Vipps</button>
                {/if}
              </div>
            </div>
          </div>
        </div>
      {/each}
    </div>
  {/if}
</main>

<style>
  main {
    width: 100%;
    max-width: 100%;
    margin: 0;
    padding: 0.5rem;
    box-sizing: border-box;
  }

  h1 {
    text-align: center;
    color: var(--text-color);
    margin: 1rem 0;
    font-size: 1.5rem;
    padding: 0 0.5rem;
  }

  .welcome-section {
    text-align: center;
    margin: 1rem 0 2rem;
    padding: 0 0.5rem;
  }

  .welcome-section h2 {
    font-size: 1.25rem;
    color: var(--text-color);
    margin-bottom: 0.5rem;
  }

  .welcome-section p {
    font-size: 0.9rem;
    color: var(--text-secondary);
    max-width: 600px;
    margin: 0 auto;
  }

  .product-list {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    padding: 0 0.5rem;
  }

  .product-card {
    display: flex;
    gap: 0.75rem;
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 0.75rem;
    background: var(--card-bg);
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    width: 100%;
    box-sizing: border-box;
  }

  .product-image {
    flex-shrink: 0;
    width: 80px;
    height: 80px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f5f5f5;
    border-radius: 4px;
    overflow: hidden;
  }

  .product-image img {
    max-width: 100%;
    max-height: 100%;
    width: auto;
    height: auto;
    object-fit: contain;
    border-radius: 4px;
  }

  .product-content {
    flex: 1;
    min-width: 0; /* Prevents flex item from overflowing */
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }

  .product-info {
    margin-bottom: 0.5rem;
  }

  .product-info h2 {
    margin: 0;
    font-size: 1rem;
    color: var(--text-color);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .description {
    font-size: 0.8rem;
    color: var(--text-secondary);
    margin: 0.25rem 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .price {
    font-size: 1rem;
    font-weight: bold;
    color: var(--price-color);
    margin: 0;
    white-space: nowrap;
  }

  .stock {
    font-size: 0.8rem;
    color: var(--text-secondary);
    margin: 0.25rem 0;
  }

  .price-action-container {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .buy-button {
    background-color: var(--button-bg);
    color: white;
    border: none;
    padding: 0.5rem 0.75rem;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.9rem;
    transition: background-color 0.3s;
    white-space: nowrap;
    min-width: 80px;
  }

  .buy-button:hover:not(:disabled) {
    background-color: var(--button-hover);
  }

  .buy-button.sold-out {
    background-color: var(--button-disabled);
    cursor: not-allowed;
  }

  .loading, .error {
    text-align: center;
    padding: 2rem;
    color: var(--text-secondary);
  }

  .error {
    color: #dc2626;
  }

  /* Desktop styles */
  @media (min-width: 768px) {
    main {
      max-width: 1200px;
      margin: 0 auto;
      padding: 2rem;
    }

    h1 {
      font-size: 2.5rem;
      margin-bottom: 2rem;
    }

    .welcome-section {
      margin: 2rem 0 3rem;
    }

    .welcome-section h2 {
      font-size: 1.5rem;
      margin-bottom: 1rem;
    }

    .welcome-section p {
      font-size: 1rem;
    }

    .product-list {
      gap: 1rem;
    }

    .product-card {
      padding: 1.5rem;
      gap: 1rem;
    }

    .product-image {
      width: 150px;
      height: 150px;
    }

    .product-info h2 {
      font-size: 1.25rem;
      white-space: normal;
    }

    .description {
      font-size: 0.9rem;
      white-space: normal;
    }

    .price {
      font-size: 1.25rem;
    }

    .stock {
      font-size: 0.9rem;
    }

    .buy-button {
      padding: 0.75rem 1rem;
      font-size: 1rem;
    }
  }
</style>
