# 🌱 EcoChain - Decentralized Carbon Credit Marketplace (I will join team from 2025)

A Web3 platform combining IoT, AI, and blockchain technology for transparent carbon credit trading.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Smart Contracts](#smart-contracts)
- [Frontend Demo](#frontend-demo)
- [API Documentation](#api-documentation)
- [Contributing](#contributing)
- [License](#license)

## 🌟 Overview

EcoChain revolutionizes the carbon credit market by providing:
- **Real-time verification** through IoT sensors
- **AI-powered validation** using satellite imagery
- **Transparent marketplace** on blockchain
- **Automated compliance** via smart contracts

## ✨ Features

- 🔗 **Smart Contract Integration**: Automated minting and burning of carbon credits
- 📡 **IoT Data Feeds**: Real-time environmental monitoring
- 🤖 **AI Verification**: Machine learning-based project validation
- 💰 **DeFi Integration**: Yield farming and liquidity pools
- 📱 **Responsive UI**: Cross-platform compatibility
- 🏛️ **DAO Governance**: Community-driven decisions

## 🛠️ Tech Stack

### Blockchain
- **Solidity** - Smart contract development
- **Hardhat** - Development framework
- **OpenZeppelin** - Security libraries
- **Polygon** - Layer 2 scaling solution

### Frontend
- **Next.js 14** - React framework
- **TypeScript** - Type safety
- **Tailwind CSS** - Styling
- **wagmi** - Web3 hooks
- **RainbowKit** - Wallet connection

### Backend
- **Node.js** - Runtime environment
- **Express.js** - Web framework
- **MongoDB** - Database
- **Chainlink** - Oracle network

### AI/ML
- **Python** - Machine learning
- **TensorFlow** - AI models
- **OpenCV** - Image processing
- **Satellite API** - Remote sensing data

## 📁 Project Structure

```
ecochain/
├── contracts/                 # Smart contracts
│   ├── CarbonCreditToken.sol
│   ├── CarbonMarketplace.sol
│   ├── ProjectVerification.sol
│   └── GovernanceToken.sol
├── frontend/                  # Next.js application
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   └── utils/
├── backend/                   # Node.js API
│   ├── routes/
│   ├── models/
│   ├── middleware/
│   └── services/
├── ai-verification/           # Python ML models
│   ├── models/
│   ├── data/
│   └── scripts/
├── iot-simulator/            # IoT data simulation
├── docs/                     # Documentation
├── scripts/                  # Deployment scripts
└── tests/                    # Test files
```

## 🚀 Installation

### Prerequisites
- Node.js (v18+)
- Python (v3.9+)
- MongoDB
- MetaMask wallet

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/ecochain.git
cd ecochain
```

### 2. Install Dependencies
```bash
# Install contract dependencies
npm install

# Install frontend dependencies
cd frontend
npm install

# Install backend dependencies
cd ../backend
npm install

# Install AI dependencies
cd ../ai-verification
pip install -r requirements.txt
```

### 3. Environment Setup
Create `.env` files in respective directories:

**Root `.env`:**
```
PRIVATE_KEY=your_private_key
POLYGON_RPC_URL=https://polygon-rpc.com
ETHERSCAN_API_KEY=your_etherscan_key
```

**Frontend `.env.local`:**
```
NEXT_PUBLIC_CONTRACT_ADDRESS=0x...
NEXT_PUBLIC_POLYGON_RPC=https://polygon-rpc.com
```

**Backend `.env`:**
```
MONGODB_URI=mongodb://localhost:27017/ecochain
JWT_SECRET=your_jwt_secret
CHAINLINK_API_KEY=your_chainlink_key
```

### 4. Deploy Contracts
```bash
npx hardhat compile
npx hardhat deploy --network polygon
```

### 5. Start Development Servers
```bash
# Terminal 1: Backend API
cd backend
npm run dev

# Terminal 2: Frontend
cd frontend
npm run dev

# Terminal 3: AI Service
cd ai-verification
python app.py
```

## 📄 Smart Contracts

### CarbonCreditToken.sol
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract CarbonCreditToken is ERC20, Ownable, ReentrancyGuard {
    struct CarbonCredit {
        uint256 amount;
        string projectId;
        uint256 vintage;
        string location;
        bool retired;
        uint256 timestamp;
    }
    
    mapping(uint256 => CarbonCredit) public credits;
    mapping(address => bool) public verifiedProjects;
    mapping(string => uint256) public projectCredits;
    
    uint256 public nextCreditId = 1;
    
    event CreditMinted(uint256 indexed creditId, string projectId, uint256 amount);
    event CreditRetired(uint256 indexed creditId, address indexed retiree);
    event ProjectVerified(address indexed project, string projectId);
    
    constructor() ERC20("EcoChain Carbon Credit", "ECC") {}
    
    function mintCredit(
        address to,
        uint256 amount,
        string memory projectId,
        uint256 vintage,
        string memory location
    ) external onlyOwner returns (uint256) {
        require(verifiedProjects[msg.sender] || msg.sender == owner(), "Not verified project");
        
        uint256 creditId = nextCreditId++;
        
        credits[creditId] = CarbonCredit({
            amount: amount,
            projectId: projectId,
            vintage: vintage,
            location: location,
            retired: false,
            timestamp: block.timestamp
        });
        
        projectCredits[projectId] += amount;
        _mint(to, amount);
        
        emit CreditMinted(creditId, projectId, amount);
        return creditId;
    }
    
    function retireCredit(uint256 creditId) external {
        require(credits[creditId].amount > 0, "Credit does not exist");
        require(!credits[creditId].retired, "Credit already retired");
        require(balanceOf(msg.sender) >= credits[creditId].amount, "Insufficient balance");
        
        credits[creditId].retired = true;
        _burn(msg.sender, credits[creditId].amount);
        
        emit CreditRetired(creditId, msg.sender);
    }
    
    function verifyProject(address project, string memory projectId) external onlyOwner {
        verifiedProjects[project] = true;
        emit ProjectVerified(project, projectId);
    }
}
```

### CarbonMarketplace.sol
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "./CarbonCreditToken.sol";

contract CarbonMarketplace is ReentrancyGuard {
    CarbonCreditToken public carbonToken;
    
    struct Listing {
        address seller;
        uint256 amount;
        uint256 pricePerCredit;
        uint256 vintage;
        string projectId;
        bool active;
        uint256 timestamp;
    }
    
    mapping(uint256 => Listing) public listings;
    uint256 public nextListingId = 1;
    uint256 public marketplaceFee = 250; // 2.5%
    
    event ListingCreated(uint256 indexed listingId, address seller, uint256 amount, uint256 price);
    event ListingSold(uint256 indexed listingId, address buyer, uint256 amount);
    event ListingCancelled(uint256 indexed listingId);
    
    constructor(address _carbonToken) {
        carbonToken = CarbonCreditToken(_carbonToken);
    }
    
    function createListing(
        uint256 amount,
        uint256 pricePerCredit,
        uint256 vintage,
        string memory projectId
    ) external returns (uint256) {
        require(amount > 0, "Amount must be positive");
        require(pricePerCredit > 0, "Price must be positive");
        require(carbonToken.balanceOf(msg.sender) >= amount, "Insufficient balance");
        
        uint256 listingId = nextListingId++;
        
        listings[listingId] = Listing({
            seller: msg.sender,
            amount: amount,
            pricePerCredit: pricePerCredit,
            vintage: vintage,
            projectId: projectId,
            active: true,
            timestamp: block.timestamp
        });
        
        // Transfer tokens to marketplace for escrow
        carbonToken.transferFrom(msg.sender, address(this), amount);
        
        emit ListingCreated(listingId, msg.sender, amount, pricePerCredit);
        return listingId;
    }
    
    function buyCredits(uint256 listingId, uint256 amount) external payable nonReentrant {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(amount <= listing.amount, "Insufficient credits available");
        
        uint256 totalPrice = amount * listing.pricePerCredit;
        uint256 fee = (totalPrice * marketplaceFee) / 10000;
        uint256 sellerPayment = totalPrice - fee;
        
        require(msg.value >= totalPrice, "Insufficient payment");
        
        // Update listing
        listing.amount -= amount;
        if (listing.amount == 0) {
            listing.active = false;
        }
        
        // Transfer credits to buyer
        carbonToken.transfer(msg.sender, amount);
        
        // Transfer payment to seller
        payable(listing.seller).transfer(sellerPayment);
        
        // Return excess payment
        if (msg.value > totalPrice) {
            payable(msg.sender).transfer(msg.value - totalPrice);
        }
        
        emit ListingSold(listingId, msg.sender, amount);
    }
    
    function cancelListing(uint256 listingId) external {
        Listing storage listing = listings[listingId];
        require(listing.seller == msg.sender, "Not listing owner");
        require(listing.active, "Listing not active");
        
        listing.active = false;
        
        // Return credits to seller
        carbonToken.transfer(msg.sender, listing.amount);
        
        emit ListingCancelled(listingId);
    }
    
    function getActiveListings() external view returns (uint256[] memory) {
        uint256 activeCount = 0;
        for (uint256 i = 1; i < nextListingId; i++) {
            if (listings[i].active) {
                activeCount++;
            }
        }
        
        uint256[] memory activeListings = new uint256[](activeCount);
        uint256 index = 0;
        
        for (uint256 i = 1; i < nextListingId; i++) {
            if (listings[i].active) {
                activeListings[index] = i;
                index++;
            }
        }
        
        return activeListings;
    }
}
```

## 🎨 Frontend Components

### MarketplacePage.tsx
```typescript
import React, { useState, useEffect } from 'react';
import { useContract, useContractRead, useContractWrite } from 'wagmi';
import { formatEther, parseEther } from 'viem';

interface Listing {
  id: number;
  seller: string;
  amount: bigint;
  pricePerCredit: bigint;
  vintage: number;
  projectId: string;
  active: boolean;
}

const MarketplacePage: React.FC = () => {
  const [listings, setListings] = useState<Listing[]>([]);
  const [loading, setLoading] = useState(true);

  const { data: activeListings } = useContractRead({
    address: process.env.NEXT_PUBLIC_MARKETPLACE_ADDRESS as `0x${string}`,
    abi: marketplaceABI,
    functionName: 'getActiveListings',
  });

  const { write: buyCredits } = useContractWrite({
    address: process.env.NEXT_PUBLIC_MARKETPLACE_ADDRESS as `0x${string}`,
    abi: marketplaceABI,
    functionName: 'buyCredits',
  });

  useEffect(() => {
    if (activeListings) {
      fetchListingDetails();
    }
  }, [activeListings]);

  const fetchListingDetails = async () => {
    // Fetch detailed listing information
    setLoading(false);
  };

  const handleBuyCredits = (listingId: number, amount: bigint, totalPrice: bigint) => {
    buyCredits({
      args: [listingId, amount],
      value: totalPrice,
    });
  };

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-green-500"></div>
      </div>
    );
  }

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-4xl font-bold text-center mb-8">Carbon Credit Marketplace</h1>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {listings.map((listing) => (
          <div key={listing.id} className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
            <div className="flex justify-between items-start mb-4">
              <h3 className="text-xl font-semibold text-gray-800">{listing.projectId}</h3>
              <span className="bg-green-100 text-green-800 px-2 py-1 rounded-full text-sm">
                Vintage {listing.vintage}
              </span>
            </div>
            
            <div className="space-y-2 mb-4">
              <p className="text-gray-600">
                <span className="font-medium">Amount:</span> {listing.amount.toString()} credits
              </p>
              <p className="text-gray-600">
                <span className="font-medium">Price:</span> {formatEther(listing.pricePerCredit)} ETH per credit
              </p>
              <p className="text-gray-600">
                <span className="font-medium">Seller:</span> 
                <span className="font-mono text-sm ml-1">
                  {listing.seller.slice(0, 6)}...{listing.seller.slice(-4)}
                </span>
              </p>
            </div>
            
            <button
              onClick={() => handleBuyCredits(
                listing.id, 
                listing.amount, 
                listing.amount * listing.pricePerCredit
              )}
              className="w-full bg-green-600 text-white py-2 px-4 rounded-md hover:bg-green-700 transition-colors"
            >
              Buy Credits
            </button>
          </div>
        ))}
      </div>
      
      {listings.length === 0 && (
        <div className="text-center py-12">
          <p className="text-gray-500 text-lg">No active listings available</p>
        </div>
      )}
    </div>
  );
};

export default MarketplacePage;
```

## 🐍 AI Verification Service

### ai_verification.py
```python
import tensorflow as tf
import numpy as np
import requests
from datetime import datetime
import cv2
from satellite_imagery import SatelliteAPI

class CarbonVerificationAI:
    def __init__(self):
        self.forest_model = tf.keras.models.load_model('models/forest_classification.h5')
        self.carbon_model = tf.keras.models.load_model('models/carbon_estimation.h5')
        self.satellite_api = SatelliteAPI()
    
    def verify_forest_project(self, coordinates, project_area):
        """
        Verify forest conservation project using satellite imagery
        """
        try:
            # Get satellite imagery
            imagery = self.satellite_api.get_imagery(coordinates, datetime.now())
            
            # Preprocess image
            processed_image = self.preprocess_imagery(imagery)
            
            # Classify forest coverage
            forest_coverage = self.forest_model.predict(processed_image)
            
            # Estimate carbon sequestration
            carbon_estimate = self.carbon_model.predict(
                np.array([forest_coverage, project_area]).reshape(1, -1)
            )
            
            verification_result = {
                'verified': forest_coverage > 0.8,  # 80% forest coverage threshold
                'forest_coverage': float(forest_coverage),
                'carbon_estimate_tons': float(carbon_estimate[0]),
                'confidence_score': float(np.max(forest_coverage)),
                'timestamp': datetime.now().isoformat()
            }
            
            return verification_result
            
        except Exception as e:
            return {
                'verified': False,
                'error': str(e),
                'timestamp': datetime.now().isoformat()
            }
    
    def preprocess_imagery(self, image_data):
        """
        Preprocess satellite imagery for model input
        """
        # Resize and normalize image
        image = cv2.resize(image_data, (224, 224))
        image = image.astype('float32') / 255.0
        return np.expand_dims(image, axis=0)
    
    def verify_renewable_energy(self, project_data):
        """
        Verify renewable energy project using IoT data
        """
        try:
            energy_output = project_data.get('energy_output_kwh', 0)
            efficiency_rating = project_data.get('efficiency_rating', 0)
            
            # Calculate carbon offset based on energy production
            carbon_offset = energy_output * 0.0004  # kg CO2 per kWh saved
            
            verification_result = {
                'verified': energy_output > 1000,  # Minimum 1MWh threshold
                'carbon_offset_tons': carbon_offset / 1000,
                'energy_output_kwh': energy_output,
                'efficiency_score': efficiency_rating,
                'timestamp': datetime.now().isoformat()
            }
            
            return verification_result
            
        except Exception as e:
            return {
                'verified': False,
                'error': str(e),
                'timestamp': datetime.now().isoformat()
            }

# Flask API endpoint
from flask import Flask, request, jsonify

app = Flask(__name__)
ai_verifier = CarbonVerificationAI()

@app.route('/verify/forest', methods=['POST'])
def verify_forest():
    data = request.json
    result = ai_verifier.verify_forest_project(
        data['coordinates'],
        data['project_area']
    )
    return jsonify(result)

@app.route('/verify/renewable', methods=['POST'])
def verify_renewable():
    data = request.json
    result = ai_verifier.verify_renewable_energy(data)
    return jsonify(result)

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

## 📡 IoT Simulator

### iot_simulator.js
```javascript
const mqtt = require('mqtt');
const { ethers } = require('ethers');

class IoTSimulator {
  constructor() {
    this.client = mqtt.connect('mqtt://localhost:1883');
    this.sensors = new Map();
    this.setupProvider();
  }

  setupProvider() {
    this.provider = new ethers.providers.JsonRpcProvider(
      process.env.POLYGON_RPC_URL
    );
    this.wallet = new ethers.Wallet(process.env.PRIVATE_KEY, this.provider);
  }

  // Simulate forest monitoring sensor
  simulateForestSensor(sensorId, location) {
    const sensor = {
      id: sensorId,
      location,
      type: 'forest_monitor',
      lastReading: Date.now()
    };

    this.sensors.set(sensorId, sensor);

    // Send readings every 5 minutes
    setInterval(() => {
      const reading = {
        sensorId,
        timestamp: Date.now(),
        co2_absorption: this.generateCO2Reading(),
        humidity: this.generateHumidityReading(),
        temperature: this.generateTemperatureReading(),
        tree_count: this.generateTreeCountReading(),
        soil_moisture: this.generateSoilMoistureReading()
      };

      this.publishReading('forest/sensors', reading);
      this.sendToBlockchain(reading);
    }, 5 * 60 * 1000);
  }

  // Simulate solar panel monitoring
  simulateSolarSensor(sensorId, location) {
    const sensor = {
      id: sensorId,
      location,
      type: 'solar_monitor',
      lastReading: Date.now()
    };

    this.sensors.set(sensorId, sensor);

    setInterval(() => {
      const reading = {
        sensorId,
        timestamp: Date.now(),
        power_output_kw: this.generatePowerOutput(),
        efficiency: this.generateEfficiencyReading(),
        temperature: this.generateTemperatureReading(),
        irradiance: this.generateIrradianceReading()
      };

      this.publishReading('solar/sensors', reading);
      this.sendToBlockchain(reading);
    }, 2 * 60 * 1000);
  }

  generateCO2Reading() {
    // Simulate CO2 absorption in kg/hour
    return (Math.random() * 5 + 2).toFixed(2);
  }

  generateHumidityReading() {
    return (Math.random() * 40 + 30).toFixed(1); // 30-70%
  }

  generateTemperatureReading() {
    return (Math.random() * 20 + 10).toFixed(1); // 10-30°C
  }

  generateTreeCountReading() {
    return Math.floor(Math.random() * 1000 + 500); // 500-1500 trees
  }

  generateSoilMoistureReading() {
    return (Math.random() * 50 + 25).toFixed(1); // 25-75%
  }

  generatePowerOutput() {
    // Simulate solar power output in kW
    const hour = new Date().getHours();
    if (hour < 6 || hour > 18) return 0; // No sun
    
    const peakHours = [11, 12, 13, 14]; // Peak sun hours
    const basePower = Math.random() * 50 + 10; // 10-60 kW
    
    return peakHours.includes(hour) 
      ? (basePower * 1.5).toFixed(2)
      : basePower.toFixed(2);
  }

  generateEfficiencyReading() {
    return (Math.random() * 15 + 85).toFixed(1); // 85-100% efficiency
  }

  generateIrradianceReading() {
    return (Math.random() * 800 + 200).toFixed(0); // W/m²
  }

  publishReading(topic, reading) {
    this.client.publish(topic, JSON.stringify(reading));
    console.log(`Published reading to ${topic}:`, reading);
  }

  async sendToBlockchain(reading) {
    try {
      // This would send data to a Chainlink oracle
      // For demo purposes, we'll just log it
      console.log('Sending to blockchain oracle:', {
        sensor: reading.sensorId,
        value: reading.co2_absorption || reading.power_output_kw,
        timestamp: reading.timestamp
      });
    } catch (error) {
      console.error('Error sending to blockchain:', error);
    }
  }

  startSimulation() {
    console.log('Starting IoT simulation...');
    
    // Start forest sensors
    this.simulateForestSensor('forest_001', 'Amazon Basin');
    this.simulateForestSensor('forest_002', 'Pacific Northwest');
    
    // Start solar sensors
    this.simulateSolarSensor('solar_001', 'California Solar Farm');
    this.simulateSolarSensor('solar_002', 'Texas Solar Plant');
    
    console.log('IoT sensors active. Publishing data...');
  }
}

// Start simulation
const simulator = new IoTSimulator();
simulator.startSimulation();

module.exports = IoTSimulator;
```

## 🧪 Testing

### test/CarbonCreditToken.test.js
```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("CarbonCreditToken", function () {
  let carbonToken;
  let owner;
  let project;
  let buyer;

  beforeEach(async function () {
    [owner, project, buyer] = await ethers.getSigners();
    
    const CarbonCreditToken = await ethers.getContractFactory("CarbonCreditToken");
    carbonToken = await CarbonCreditToken.deploy();
    await carbonToken.deployed();
  });

  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      expect(await carbonToken.owner()).to.equal(owner.address);
    });

    it("Should have correct name and symbol", async function () {
      expect(await carbonToken.name()).to.equal("EcoChain Carbon Credit");
      expect(await carbonToken.symbol()).to.equal("ECC");
    });
  });

  describe("Credit Minting", function () {
    it("Should mint credits for verified projects", async function () {
      await carbonToken.verifyProject(project.address, "PROJECT_001");
      
      await carbonToken.connect(project).mintCredit(
        buyer.address,
        1000,
        "PROJECT_001",
        2024,
        "Amazon Basin"
      );

      expect(await carbonToken.balanceOf(buyer.address)).to.equal(1000);
    });

    it("Should fail minting for unverified projects", async function () {
      await expect(
        carbonToken.connect(project).mintCredit(
          buyer.address,
          1000,
          "PROJECT_001",
          2024,
          "Amazon Basin"
        )
      ).to.be.revertedWith("Not verified project");
    });
  });

  describe("Credit Retirement", function () {
    beforeEach(async function () {
      await carbonToken.verifyProject(project.address, "PROJECT_001");
      await carbonToken.connect(project).mintCredit(
        buyer.address,
        1000,
        "PROJECT_001",
        2024,
        "Amazon Basin"
      );
    });

    it("Should retire credits successfully", async function () {
      await carbonToken.connect(buyer).retireCredit(1);
      
      const credit = await carbonToken.credits(1);
      expect(credit.retired).to.be.true;
      expect(await carbonToken.balanceOf(buyer.address)).to.equal(0);
    });
  });
});
```

## 📚 API Documentation

### RESTful API Endpoints

```typescript
// GET /api/projects - Get all verified projects
interface Project {
  id: string;
  name: string;
  location: string;
  type: 'forest' | 'renewable' | 'agriculture';
  totalCredits: number;
  availableCredits: number;
  verificationStatus: 'pending' | 'verified' | 'rejected';
  iotData: IoTReading[];
}

// POST /api/projects - Create new project
interface CreateProjectRequest {
  name: string;
  location: string;
  type: string;
  description: string;
  estimatedOffset: number;
  coordinates: [number, number];
}

// GET /api/marketplace/listings - Get marketplace listings
interface MarketplaceListing {
  id: number;
  seller: string;
  amount: number;
  pricePerCredit: number;
  vintage: number;
  projectId: string;
  active: boolean;
}

// POST /api/verification/ai - Submit for AI verification
interface AIVerificationRequest {
  projectId: string;
  projectType: 'forest' | 'renewable';
  coordinates?: [number, number];
  iotData?: any;
}
```

## 🚀 Deployment

### Deploy to Polygon Mainnet
```bash
# Compile contracts
npx hardhat compile

# Deploy to Polygon
npx hardhat run scripts/deploy.js --network polygon

# Verify contracts
npx hardhat verify --network polygon DEPLOYED_CONTRACT_ADDRESS

# Deploy frontend to Vercel
vercel --prod

# Deploy backend to Heroku
git push heroku main
```

### Environment Variables for Production
```bash
# Smart Contracts
POLYGON_RPC_URL=https://polygon-rpc.com
PRIVATE_KEY=your_private_key
ETHERSCAN_API_KEY=your_api_key

# Frontend
NEXT_PUBLIC_CONTRACT_ADDRESS=0x...
NEXT_PUBLIC_MARKETPLACE_ADDRESS=0x...
NEXT_PUBLIC_POLYGON_RPC=https://polygon-rpc.com

# Backend
MONGODB_URI=mongodb+srv://...
JWT_SECRET=your_jwt_secret
CHAINLINK_API_KEY=your_chainlink_key
SATELLITE_API_KEY=your_satellite_api_key

# AI Service
TENSORFLOW_MODEL_PATH=./models/
SATELLITE_API_URL=https://api.satellite.com
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🎯 Hackathon Submission

This project was developed for [Hackathon Name] and demonstrates:

- ✅ **Innovation**: Combining IoT, AI, and blockchain for carbon credits
- ✅ **Technical Excellence**: Full-stack Web3 application with smart contracts
- ✅ **Real-world Impact**: Addressing climate change through technology
-
