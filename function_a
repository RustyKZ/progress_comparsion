# Django/DRF
@csrf_exempt
def user_deposit_silver_usdt(request):
    try:
        data = json.loads(request.body)
        print(f'USER DEPOSIT SILVER: incoming data - {data}')
        player_id = data['user_id']
        token = data['token']        
        usdt_value = data['usdt_value']
        transaction_hash = data['transaction_hash']
        ip_address = data['ip_address']
        player = Players.objects.get(id=player_id)
        token_settings = TokenSettings.objects.get(id=1)
        payment_settings = DepositWithdrawSettings.objects.get(id=1)
        rate_usd = payment_settings.silver_usd_rate
        usdt_contract = '0x55d398326f99059fF775485246999027B3197955'
        current_time = datetime.utcnow().replace(tzinfo=timezone.utc)
        unix_time = int(current_time.timestamp())
        try:            
            transaction_data = get_transaction_data_usdt(transaction_hash)            
            token_value = int(math.floor(transaction_data['amount'] / 10**18))            
            if transaction_data['status'] and transaction_data['from'].lower() == player.wallet.lower() and transaction_data['to'].lower() == token_settings.host_wallet.lower() and transaction_data['contract'].lower() == usdt_contract.lower() and token_value == usdt_value:
                try:
                    transaction_log = TransactionsLog.objects.get(transaction_hash=transaction_hash)
                    transaction_error = TransactionsError(
                        date = current_time,
                        user_id = player_id,
                        sender = player.wallet,
                        transaction_hash = transaction_hash,
                        amount = usdt_value,
                        ip_address = ip_address,
                        type = 0, #Silvercoin
                        message = 'USER DEPOSIT SILVERCOIN (USDT): Transaction dublicated'
                    )
                    transaction_error.save()
                    return JsonResponse({'status': False, 'message': 'USER DEPOSIT SILVERCOIN (USDT): transaction already exists', 'error': 483})
                except Exception as transaction_not_found:                    
                    player.silvercoin += usdt_value * rate_usd
                    player.save()                    
                    player_data = get_player_data(player.id)
                    action = {
                        "date": unix_time,
                        "coin": "silvercoin",
                        "action": "deposit",
                        "value": usdt_value * rate_usd,
                        "method": "self",
                        "transaction_hash": transaction_hash,
                        "ip_address": ip_address
                    }
                    player_data.coin_activity.append(action)
                    player_data.history_silver.append(action)
                    player_data.save()                    
                    transaction_log = TransactionsLog(
                        transaction_hash = transaction_hash,
                        sender = transaction_data['from'],
                        recipient = transaction_data['to'],
                        date = current_time,
                        contract = transaction_data['contract'],
                        amount = token_value
                    )
                    transaction_log.save()
                    
                    try:
                        referer = Players.objects.get(id=player.referer_id)
                        if referer:
                            ref_data = get_player_data(referer.id)
                            ref_data.ref_silver += math.floor(usdt_value * rate_usd / 10)
                            ref_data.ref_bonus += math.floor(usdt_value * rate_usd / 100)
                            ref_action_silver = {
                                "date": unix_time,
                                "coin": "silvercoin",
                                "action": "deposit",
                                "value": math.floor(usdt_value * rate_usd / 10),
                                "method": "referal",
                                "transaction_hash": transaction_hash
                            }
                            ref_action_bonus = {
                                "date": unix_time,
                                "coin": "silvercoin",
                                "action": "deposit",
                                "value": math.floor(usdt_value * rate_usd / 100),
                                "method": "referal",
                                "transaction_hash": transaction_hash
                            }
                            ref_data.coin_activity.append(ref_action_silver)
                            ref_data.history_silver.append(ref_action_silver)
                            ref_data.coin_activity.append(ref_action_bonus)
                            ref_data.history_bonus.append(ref_action_bonus)
                            ref_data.save()
                            print('Referer got silvercoins!')
                    except:
                        print('Referer not found!')
                        pass
                    return JsonResponse({'status': True, 'message': 'USER DEPOSIT SILVERCOIN: - OK', 'code': 488})
            else:
                transaction_error = TransactionsError(
                    date = current_time,
                    user_id = player_id,
                    sender = player.wallet,
                    transaction_hash = transaction_hash,
                    amount = usdt_value,
                    ip_address = ip_address,
                    type = 0, #Silvercoin
                    message = 'USER DEPOSIT SILVERCOIN (USDT): Transaction not verified'
                )
                transaction_error.save()
                return JsonResponse({'status': False, 'message': 'USER DEPOSIT SILVERCOIN (USDT): Transaction not verified', 'error': 484})        
        except Exception as e:
            print(f'USER DEPOSIT SILVERCOIN (USDT) ERROR - players data: {e}')
            error_message = str(e)
            transaction_error = TransactionsError(
                date = current_time,
                user_id = player_id,
                sender = player.wallet,
                transaction_hash = transaction_hash,
                amount = usdt_value,
                ip_address = ip_address,
                type = 0, #Silvercoin
                message = 'USER DEPOSIT SILVERCOIN (USDT): Transaction not found - ' + error_message
            )
            transaction_error.save()
            return JsonResponse({'status': False, 'message': 'USER DEPOSIT SILVERCOIN: Transaction not found', 'error': 484})
    except Exception as e:
        print(f'USER DEPOSIT SILVERCOIN (USDT): {e}')
        error_message = str(e)
        transaction_error = TransactionsError(
            date = current_time,
            user_id = player_id,
            sender = player.wallet,
            transaction_hash = transaction_hash,
            amount = usdt_value,
            ip_address = ip_address,
            type = 0, #Silvrcoin
            message = 'USER DEPOSIT SILVERCOIN (USDT): unkmown error - ' + error_message
        )
        transaction_error.save()
        return JsonResponse({'status': False, 'message': 'USER DEPOSIT SILVERCOIN (USDT): unkmown error', 'error': 484})
