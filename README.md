/// <summary>
        /// Retrieves (default from list) accounts list if there are no filters mentioned 
        /// Retrieves TO list for the mentioned account index in the query
        /// </summary>
        /// <param name="user"></param>
        /// <param name="filter"></param>
        /// <returns>List of transaction (MCD) accounts</returns>
        [ProducesResponseType(typeof(List<Account>), StatusCodes.Status200OK)]
        [Route("transactionaccounts")]
        [HttpGet]
        public IActionResult GetTransactionAccounts([MtbToken, Required] User user, [FromQuery] AccountsFilter filter)
        {
            filter.AccountListType = AccountListType.To;
            filter.Module = Module.CheckDeposit;

            return Ok(_accountsService.GetTransactionAccounts(user, filter));
        }




public List<Account> GetTransactionAccounts(User user, AccountsFilter filter)
        {
            var accounts = _cbsAccountsProvider.GetAccounts(user, filter);

            switch (filter.Module)
            {
                case Module.CheckDeposit:
                    if (filter.AccountListType == AccountListType.To)
                    {
                        var checkingAccounts = _accountsMapper.MapCheckingAccounts(accounts
                            .Where(x => ProductCode.CheckingAccounts.Contains(x.ProductCode.ToUpper())));

                        return _accountsMapper.MapTransactionAccounts(checkingAccounts);
                    }
                    break;
            }

            throw new BadRequestException(ErrorCode.InvalidRequest, nameof(ErrorCode.InvalidRequest).GetUserDescription<ErrorCode>(), nameof(ErrorCode.InvalidRequest).GetDeveloperDescription<ErrorCode>());
        }

 

