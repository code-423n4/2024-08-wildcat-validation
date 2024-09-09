https://github.com/code-423n4/2024-08-wildcat/blob/main/src/WildcatSanctionsEscrow.sol#L43-L43

		  function releaseEscrow() public override {
			if (!canReleaseEscrow()) revert CanNotReleaseEscrow();

			uint256 amount = balance();
			address _account = account;
			address _asset = asset;

			asset.safeTransfer(_account, amount);

			emit EscrowReleased(_account, _asset, amount);
		  }

while emit in function releaseEscrow(), should include borrower, as following:

    emit EscrowReleased(borrower, _account, _asset, amount);