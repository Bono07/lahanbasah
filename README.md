# lahanbasah
project/
├── src/
│   ├── main.rs       # Entry point
│   ├── lib.rs        # Library code (if any)
├── Cargo.toml        # Dependencies and project configuration

[dependencies]
web3 = "0.15"

// src/main.rs

use std::collections::HashMap;
use web3::types::{Address, U256};
use web3::Transport;

// Function to perform an airdrop to a list of addresses
fn perform_airdrop<T: Transport>(web3: &web3::Web3<T>, airdrop_amount: U256, addresses: Vec<Address>) -> Result<(), web3::error::Error> {
    let sender_account: Address = "YOUR_SENDER_ADDRESS".parse().unwrap();  // Replace with your sender address
    let private_key: &str = "YOUR_PRIVATE_KEY";  // Replace with your private key

    let mut tx_hashes = Vec::new();
    
    for &address in addresses.iter() {
        let tx = web3.eth().send_transaction(web3::types::TransactionRequest {
            from: sender_account,
            to: Some(address),
            gas: None,
            gas_price: None,
            value: Some(airdrop_amount),
            data: None,
            nonce: None,
            condition: None,
        }).map_err(|e| {
            eprintln!("Failed to send transaction: {:?}", e);
            e
        })?;
        
        tx_hashes.push(tx);
    }

    println!("Airdrop successful. Transaction hashes: {:?}", tx_hashes);
    Ok(())
}

fn main() {
    let http = web3::transports::Http::new("http://localhost:8545").unwrap();  // Replace with your RPC endpoint
    let web3 = web3::Web3::new(http);

    let addresses = vec![
        "ADDRESS_1".parse().unwrap(),
        "ADDRESS_2".parse().unwrap(),
        // Add more addresses as needed
    ];

    let airdrop_amount = U256::from(100);  // Amount to airdrop in Wei

    if let Err(e) = perform_airdrop(&web3, airdrop_amount, addresses) {
        eprintln!("Error: {:?}", e);
    }
}

[dependencies]
actix-web = "4.0"

// src/main.rs

use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use std::sync::Mutex;

// In-memory storage for simplicity (not suitable for production)
struct AppState {
    balances: Mutex<HashMap<String, u32>>,
}

async fn faucet(info: web::Path<String>, state: web::Data<AppState>) -> impl Responder {
    let mut balances = state.balances.lock().unwrap();
    let balance = balances.entry(info.into_inner()).or_insert(100);
    *balance += 1;
    HttpResponse::Ok().body(format!("New balance: {}", balance))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let state = web::Data::new(AppState {
        balances: Mutex::new(HashMap::new()),
    });

    HttpServer::new(move || {
        App::new()
            .app_data(state.clone())
            .service(web::resource("/{user_id}").route(web::get().to(faucet)))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}

