# Fast API
async def player_buy_gamecoins_by_crypto(user_tg_id: int, gamecoin_value: int, buyer_address_raw: str, coin_value: Decimal, coin_type: str, transaction_boc: str) -> dict:
    async with async_session() as session:
        try:
            current_time = int(datetime.now(timezone.utc).timestamp())

            description = {
                "user_tg_id": user_tg_id,
                "type": "transaction",
                "action": "player_buy_gamecoins_by_crypto",
                "asset_type": "gamecoin",
                "asset_amount": gamecoin_value,
                "date": current_time,
                "currency_type": coin_type.upper(),
                "currency_amount": str(coin_value)
            }

            if coin_type.upper() == "BMT":
                transaction_result = await checking_transaction_bmt(user_tg_id, buyer_address_raw, coin_value, transaction_boc, description)
            else:
                transaction_result = await checking_transaction_ton(user_tg_id, buyer_address_raw, coin_value, transaction_boc, description)

            if not transaction_result['status']:
                return {"status": False, "notify_code": "notify_buy_gamecoins_error", "message": "Transaction is not confirmed"}
            else:
                if coin_type.upper() == "BMT":
                    transaction_query = select(JettonTransactions).filter(JettonTransactions.transaction_hash == transaction_result['transaction_hash'])
                    transactions = await session.execute(transaction_query)
                    transaction = transactions.scalars().first()
                else:
                    transaction_query = select(TonTransactions).filter(TonTransactions.transaction_hash == transaction_result['transaction_hash'])
                    transactions = await session.execute(transaction_query)
                    transaction = transactions.scalars().first()
            
            # Получаем игрока
            player_query = select(Players).filter(Players.tg_id == user_tg_id)
            players = await session.execute(player_query)
            player = players.scalars().first()
            if not player:
                transaction.not_shipped = True
                transaction.not_shipped_reason = {
                    "function": "player_buy_gamecoins_by_crypto",
                    "error": "Player not found"
                }
                flag_modified(transaction, "not_shipped_reason")
                await session.commit()
                return {"status": False, "notify_code": "notify_buy_mgr_lvl_error", "message": "Player not found"}
                                                
            design = await get_latest_game_design()
            gk_1000_bmt_rate = getattr(design, "currency_rate_1kgc_bmt", DEFAULT_CURRENCY_RATE_1K_GC_BMT)

            gamecoins_total_cost = math.floor(gamecoin_value // 1000 * gk_1000_bmt_rate)

            cheat_coin_value = False
            if coin_type.upper() == "BMT":
                if gamecoins_total_cost > coin_value:
                    cheat_coin_value = True
            else:
                currency_rate = await get_currency_rate_from_celery_data()
                ton_rate = currency_rate.get("ton", None)
                bmt_rate = currency_rate.get("bmt", None)
                if ton_rate and bmt_rate:
                    if gamecoins_total_cost > coin_value * ton_rate/bmt_rate:
                        cheat_coin_value = True
            if cheat_coin_value:
                transaction.not_shipped = True
                transaction.not_shipped_reason = {
                    "function": "player_buy_gamecoins_by_crypto",
                    "error": "Suspicion of hacker activity"
                }
                flag_modified(transaction, "not_shipped_reason")

                # Logging suspicious activity ------------------------------------- Logging suspicious activity 

                await session.commit()
                return {"status": False, "notify_code": "notify_buy_gamecoins_error", "message": "Suspicion of hacker activity"}
            
            player.coins += gamecoin_value
            player.last_activity = current_time
            await session.commit()
            return {"status": True}
        
        except Exception as e:
            await session.rollback()
            logger.error(f"async FUNCTION player_buy_gamecoins_by_crypto - Exception: \n{e}", user_tg_id=user_tg_id)
            return {"status": False, "notify_code": "notify_buy_gamecoins_error", "message": "An error occurred while trying to upgrade player's manager level"}
