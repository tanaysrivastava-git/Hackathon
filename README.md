# Hackathon

// src/app/services/blockchain.service.ts
import { Injectable } from '@angular/core';
import { ethers } from 'ethers';
import tokenAbi from '../../assets/ConsortiumToken.json';

@Injectable({ providedIn: 'root' })
export class BlockchainService {
  provider!: ethers.BrowserProvider;
  signer!: ethers.Signer;
  token!: ethers.Contract;

  contractAddress = 'PASTE_DEPLOYED_ADDRESS_HERE';

  async connect() {
    this.provider = new ethers.BrowserProvider((window as any).ethereum);
    await this.provider.send('eth_requestAccounts', []);
    this.signer = await this.provider.getSigner();

    this.token = new ethers.Contract(
      this.contractAddress,
      tokenAbi.abi,
      this.signer
    );
  }

  async balanceOf(addr: string) {
    return ethers.formatUnits(
      await this.token.balanceOf(addr),
      18
    );
  }

  async transfer(to: string, amount: string) {
    return await this.token.transfer(
      to,
      ethers.parseUnits(amount, 18)
    );
  }

  async mint(to: string, amount: string) {
    return await this.token.mint(
      to,
      ethers.parseUnits(amount, 18)
    );
  }
}



// dashboard.component.ts
import { Component } from '@angular/core';
import { BlockchainService } from '../services/blockchain.service';

@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html',
})
export class DashboardComponent {
  address = '';
  balance = '';

  constructor(private bc: BlockchainService) {}

  async connect() {
    await this.bc.connect();
    this.address = await this.bc.signer.getAddress();
    this.balance = await this.bc.balanceOf(this.address);
  }

  async mint(amount: string) {
    await this.bc.mint(this.address, amount);
    this.balance = await this.bc.balanceOf(this.address);
  }

  async transfer(to: string, amount: string) {
    await this.bc.transfer(to, amount);
    this.balance = await this.bc.balanceOf(this.address);
  }
}



<!-- dashboard.component.html -->
<button (click)="connect()">Connect Wallet</button>

<p>Address: {{ address }}</p>
<p>Balance: {{ balance }} CS</p>

<input #amt placeholder="Mint Amount">
<button (click)="mint(amt.value)">Mint (Bank)</button>

<hr>

<input #to placeholder="Receiver">
<input #amt2 placeholder="Amount">
<button (click)="transfer(to.value, amt2.value)">Send CS</button>







