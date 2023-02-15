# BNB-GALAXY-
BNB GALAXY FARMING 
// SPDX-License-Identifier: MIT
pragma solidity 0.8.0; 

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/math/SafeMath.sol"; 

contract BNBGalaxyV1FarmingSmartContract {
    using SafeMath for uint256;
    IERC20 public bnbToken = IERC20(0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c); 

    address public marketingWallet;
    uint256 public constant depositFeeRate = 12; // 1.2%
    uint256 public constant withdrawFeeRate = 12; // 1.2%
    uint256 public constant marketingFeeRate = 100; // 10% 

    event Transfer(address indexed from, address indexed to, uint256 value); 

    mapping(address => uint256) public balanceOf;
    mapping(address => uint256) public lockedAmount;
    mapping(address => bool) public hasPaidMarketingFee;
    mapping(address => uint256) public lastWithdrawTime;
    uint256 public totalSupply; 

    constructor() {
        totalSupply = 1000000000;
        balanceOf[msg.sender] = totalSupply;
        marketingWallet = 0x226f2CfDF0D380FA4Fd84B7Df5Ae6950A8Bbd0Fa;
    } 

    function deposit(uint256 _value) public {
        uint256 depositFee = _value.mul(depositFeeRate).div(1000);
        uint256 depositAmount = _value.sub(depositFee);
        require(bnbToken.transferFrom(msg.sender, address(this), depositAmount), "BNB transfer failed");
        balanceOf[msg.sender] += depositAmount;
        if (!hasPaidMarketingFee[msg.sender]) {
            uint256 marketingFee = depositAmount.mul(marketingFeeRate).div(10000);
            require(bnbToken.transferFrom(msg.sender, marketingWallet, marketingFee), "Marketing fee transfer failed");
            hasPaidMarketingFee[msg.sender] = true;
        }
    } 

    function lock(uint256 _value) public {
        require(balanceOf[msg.sender] >= _value, "Not enough balance.");
        balanceOf[msg.sender] -= _value;
        require(bnbToken.approve(address(this), _value), "BNB approve failed");
        lockedAmount[msg.sender] += _value;
        lastWithdrawTime[msg.sender] = block.timestamp;
    } 

    function withdraw(uint256 _value) public {
        uint256 elapsed = block.timestamp - lastWithdrawTime[msg.sender];
        uint256 dailyInterest = (lockedAmount[msg.sender] * 3 / 365) * elapsed / 1 days;
        uint256 withdrawAmount = _value + dailyInterest;
        uint256 withdrawFee = withdrawAmount.mul(withdrawFeeRate).div(1000);
        uint256 actualWithdrawAmount = withdrawAmount.sub(withdrawFee);
        require(lockedAmount[msg.sender] >= actualWithdrawAmount, "Not enough locked balance.");
        lockedAmount[msg.sender] -= actualWithdrawAmount;
        require(bnbToken.transfer(msg.sender, actualWithdrawAmount), "BNB transfer failed");
        balanceOf[msg.sender] += _value;
        lastWithdrawTime[msg.sender] = block.timestamp; 

        if (balanceOf[msg.sender] >= _value) {
            require(balanceOf[msg.sender] >= _value, "Not enough balance.");
            // Resto del código
        }
    } 

    function transfer(address _to, uint256 _value) public {
    require(_to != address(0), "Cannot transfer to the zero address");
    require(balanceOf[msg.sender] >= _value, "Not enough balance.");
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
    emit Transfer(msg.sender, _to, _value);
} 

}
