# NFT-Staking-with-Rewards
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract NFTStaking is Ownable {
    IERC721 public nftContract;
    IERC20 public rewardToken;

    struct StakeInfo {
        uint256 tokenId;
        uint256 stakedAt;
    }

    mapping(address => StakeInfo) public stakedNFTs;

    event Staked(address owner, uint256 tokenId);
    event Unstaked(address owner, uint256 tokenId, uint256 reward);

    constructor(address _nft, address _reward) {
        nftContract = IERC721(_nft);
        rewardToken = IERC20(_reward);
    }

    function stake(uint256 tokenId) external {
        require(stakedNFTs[msg.sender].tokenId == 0, "Already staked");
        nftContract.transferFrom(msg.sender, address(this), tokenId);
        stakedNFTs[msg.sender] = StakeInfo(tokenId, block.timestamp);
        emit Staked(msg.sender, tokenId);
    }

    function unstake() external {
        StakeInfo memory stakeInfo = stakedNFTs[msg.sender];
        require(stakeInfo.tokenId != 0, "No NFT staked");

        uint256 reward = (block.timestamp - stakeInfo.stakedAt) * 1 ether / 1 days; // 1 token per day

        nftContract.transferFrom(address(this), msg.sender, stakeInfo.tokenId);
        delete stakedNFTs[msg.sender];

        rewardToken.transfer(msg.sender, reward);
        emit Unstaked(msg.sender, stakeInfo.tokenId, reward);
    }
}
