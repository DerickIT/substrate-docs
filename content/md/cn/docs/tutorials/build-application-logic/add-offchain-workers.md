---
title: 添加链下工作程序
description: 说明如何修改模块以包含链下工作程序，以及如何从链下工作程序提交事务以更新链上状态。
keywords:
  - 链下工作程序
  - ocw
  - 事务
  - 链下数据
  - 已签名事务
  - 未签名事务
  - 已签名有效负载
---

<div class="warning">
	 请参考<a href="https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/frame_offchain_workers/index.html">Rust 文档</a>以获取有关此主题的最新文档。
</div>

本教程说明了如何修改模块以包含链下工作程序，以及如何配置模块和运行时以使链下工作程序能够提交更新链上状态的事务。

## 使用链下工作程序

如果您使用链下工作程序执行长时间运行的计算或从离线源获取数据，则您很可能希望将这些操作的结果存储在链上。
但是，链下存储与链上资源是分开的，您不能将链下工作程序处理的数据直接保存到链上存储。
要将链下工作程序处理的任何数据存储为链上状态的一部分，您必须创建事务以将数据从链下工作程序存储发送到链上存储系统。

本教程说明了如何创建具有发送已签名或未签名事务以将链下数据存储在链上的能力的链下工作程序。
通常，已签名事务更安全，但需要调用帐户来处理交易费用。
例如：

- 如果您想记录关联的事务调用者帐户并从调用者帐户中扣除交易费用，请使用**已签名事务**。
- 如果您想记录关联的事务调用者，但不想让调用者负责支付交易费用，请使用**未签名事务和已签名有效负载**。

## 使用未签名事务

也可以提交没有已签名有效负载的**未签名事务**——例如，因为您根本不想记录关联的事务调用者。
但是，允许未签名事务修改链状态存在很大的风险。
未签名事务代表恶意用户可能利用的潜在攻击媒介。
如果您要允许链下工作程序发送未签名事务，则应包含确保事务已授权的逻辑。
有关如何使用链上状态验证未签名事务的示例，请参阅[`enact_authorized_upgrade`](https://github.com/paritytech/polkadot-sdk/blob/master/cumulus/pallets/parachain-system/src/lib.rs)调用中的`ValidateUnsigned`实现。
在此示例中，该调用通过验证给定的代码哈希是否先前已授权来验证未签名事务。

同样重要的是要考虑，即使是带有已签名有效负载的未签名事务也可能被利用，因为除非您实现严格的逻辑来检查事务的有效性，否则不能假设链下工作程序是可靠的来源。
在大多数情况下，在写入存储之前检查事务是否由链下工作程序提交不足以保护网络。
不要假设链下工作程序在没有安全措施的情况下是可信的，而应有意设置限制性权限，以限制对该过程及其可以执行的操作的访问。

请记住，未签名事务本质上是进入运行时的**敞开的门**。
您应该仅在仔细考虑允许它们执行的条件后才使用它们。
如果没有安全措施，恶意行为者可能会模仿链下工作程序并访问运行时存储。

## 开始之前

在开始之前，请验证以下内容：

- 您已通过安装 [Rust 和 Rust 工具链](/install/) 为 Substrate 开发配置了您的环境。

- 您已完成[构建本地区块链](/tutorials/build-a-blockchain/build-local-blockchain/)教程，并在本地安装了来自开发者中心的 Substrate 节点模板。

- 您熟悉如何使用 FRAME 宏和编辑模块的逻辑。

- 您熟悉如何在运行时修改模块的配置特征。

## 教程目标

通过完成本教程，您将能够：

- 确定使用未签名事务所涉及的风险。
- 向模块添加链下工作程序函数。
- 配置模块和运行时以使链下工作程序能够提交已签名事务。
- 配置模块和运行时以使链下工作程序能够提交未签名事务。
- 配置模块和运行时以使链下工作程序能够提交带有已签名有效负载的未签名事务。

## 已签名事务

要提交已签名事务，您必须配置您的模块和运行时以至少启用一个帐户供链下工作程序使用。
在较高的层次上，配置模块以使用办公室链工作程序并提交已签名事务涉及以下步骤：

- [在模块中配置链下工作程序](#configure-the-offchain-worker-in-the-pallet)。
- [在运行时实现模块和所需的特征](#implement-the-pallet-in-the-runtime)。
- [添加用于签名事务的帐户](#add-an-account-for-signing-transactions)。

### 在模块中配置链下工作程序

要启用链下工作程序发送已签名事务：

1. 在文本编辑器中打开模块的 `src/lib.rs` 文件。
2. 将 `#[pallet::hooks]` 宏和链下工作程序的入口点添加到代码中。

   例如：

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	/// Offchain worker entry point.
   	///
   	/// By implementing `fn offchain_worker` you declare a new offchain worker.
   	/// This function will be called when the node is fully synced and a new best block is
   	/// successfully imported.
   	/// Note that it's not guaranteed for offchain workers to run on EVERY block, there might
   	/// be cases where some blocks are skipped, or for some the worker runs twice (re-orgs),
   	/// so the code should be able to handle that.
   	fn offchain_worker(block_number: T::BlockNumber) {
   		log::info!("Hello from pallet-ocw.");
   		// The entry point of your code called by offchain worker
   	}
   	// ...
   }
   ```

3. 添加 `offchain_worker` 函数的逻辑。
4. 将[`CreateSignedTransaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.CreateSignedTransaction.html)添加到模块的 `Config` 特征中。
   例如，您的模块 `Config` 特征应如下所示：

   ```rust
   /// This pallet's configuration trait
   #[pallet::config]
   pub trait Config: CreateSignedTransaction<Call<Self>> + frame_system::Config {
   	// ...
   }
   ```

5. 向模块 `Config` 特征添加 `AuthorityId` 类型：

   ```rust
   #[pallet::config]
   pub trait Config: CreateSignedTransaction<Call<Self>> + frame_system::Config {
   	// ...
   type AuthorityId: AppCrypto<Self::Public, Self::Signature>;
   }
   ```

6. 添加一个带有 `sr25519` 签名密钥的 `crypto` 模块，以确保您的模块拥有一个可用于签名事务的帐户。

   ```rust
   use sp_core::{crypto::KeyTypeId};

   // ...

   pub const KEY_TYPE: KeyTypeId = KeyTypeId(*b"demo");

   // ...

   pub mod crypto {
   	use super::KEY_TYPE;
   	use sp_core::sr25519::Signature as Sr25519Signature;
   	use sp_runtime::{
   		app_crypto::{app_crypto, sr25519},
   		traits::Verify, MultiSignature, MultiSigner
   	};
   	app_crypto!(sr25519, KEY_TYPE);

   	pub struct TestAuthId;

   	// implemented for runtime
   	impl frame_system::offchain::AppCrypto<MultiSigner, MultiSignature> for TestAuthId {
   	type RuntimeAppPublic = Public;
   	type GenericSignature = sp_core::sr25519::Signature;
   	type GenericPublic = sp_core::sr25519::Public;
   	}
   }
   ```

   [`app_crypto` 宏](https://paritytech.github.io/substrate/master/sp_application_crypto/macro.app_crypto.html)声明一个具有 `sr25519` 签名的帐户，该帐户由`KEY_TYPE`标识。
	 在此示例中，`KEY_TYPE`为`demo`。
   请注意，此宏不会创建新帐户。
   该宏只是声明此模块可以使用`crypto`帐户。

7. 初始化链下工作程序要使用的帐户，以向链上存储发送已签名事务。

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
   	let signer = Signer::<T, T::AuthorityId>::all_accounts();

   	// ...
   }
   ```

   此代码使您可以检索此模块拥有的所有签名者。

8. 使用 `send_signed_transaction()` 创建已签名事务调用：

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
   	let signer = Signer::<T, T::AuthorityId>::all_accounts();

   	// Using `send_signed_transaction` associated type we create and submit a transaction
   	// representing the call we've just created.
   	// `send_signed_transaction()` return type is `Option<(Account<T>, Result<(), ()>)>`. It is:
   	//	 - `None`: no account is available for sending transaction
   	//	 - `Some((account, Ok(())))`: transaction is successfully sent
   	//	 - `Some((account, Err(())))`: error occurred when sending the transaction
   	let results = signer.send_signed_transaction(|_account| {
   		Call::on_chain_call { key: val }
   	});

   	// ...
   }
   ```

9. 通过检查返回的 `results` 来检查事务是否已成功提交到链上并执行正确的错误处理。

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
   	// ...

   	for (acc, res) in &results {
   		match res {
   			Ok(()) => log::info!("[{:?}]: submit transaction success.", acc.id),
   			Err(e) => log::error!("[{:?}]: submit transaction failure. Reason: {:?}", acc.id, e),
   		}
   	}

   	Ok(())
   }
   ```

### 在运行时实现模块

1. 在文本编辑器中打开节点模板的 `runtime/src/lib.rs` 文件。

1. 将 `AuthorityId` 添加到模块的配置中，并确保它使用 `crypto` 模块中的 `TestAuthId`：

	 ```rust
   impl pallet_your_ocw_pallet::Config for Runtime {
	   // ...
	   type AuthorityId = pallet_your_ocw_pallet::crypto::TestAuthId;
   }
   ```

1. 在运行时实现 `CreateSignedTransaction` 特征。

   因为您已将模块配置为实现 `CreateSignedTransaction` 特征，所以您还需要为运行时实现该特征。

   通过查看[`CreateSignedTransaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.CreateSignedTransaction.html)，您可以看到您只需要为运行时实现 `create_transaction()` 函数。
   例如：

   ```rust
   use codec::Encode;
   use sp_runtime::{generic::Era, SaturatedConversion};

   // ...

   impl<LocalCall> frame_system::offchain::CreateSignedTransaction<LocalCall> for Runtime
   where
	    RuntimeCall: From<LocalCall>,
   {
	    fn create_transaction<C: frame_system::offchain::AppCrypto<Self::Public, Self::Signature>>(
   		   call: RuntimeCall,
	       public: <Signature as Verify>::Signer,
		     account: AccountId,
		     nonce: Nonce,
	     ) -> Option<(RuntimeCall, <UncheckedExtrinsic as traits::Extrinsic>::SignaturePayload)> {
		     let tip = 0;
		     // take the biggest period possible.
		     let period =
			      BlockHashCount::get().checked_next_power_of_two().map(|c| c / 2).unwrap_or(2) as u64;
		     let current_block = System::block_number()
			      .saturated_into::<u64>()
			      // The `System::block_number` is initialized with `n+1`,
			      // so the actual block number is `n`.
			      .saturating_sub(1);
		     let era = Era::mortal(period, current_block);
		     let extra = (
			      frame_system::CheckNonZeroSender::<Runtime>::new(),
			      frame_system::CheckSpecVersion::<Runtime>::new(),
			      frame_system::CheckTxVersion::<Runtime>::new(),
			      frame_system::CheckGenesis::<Runtime>::new(),
			      frame_system::CheckEra::<Runtime>::from(era),
			      frame_system::CheckNonce::<Runtime>::from(nonce),
			      frame_system::CheckWeight::<Runtime>::new(),
			      pallet_transaction_payment::ChargeTransactionPayment::<Runtime>::from(tip),
		     );
		     let raw_payload = SignedPayload::new(call, extra)
			      .map_err(|e| {
				       log::warn!("Unable to create signed payload: {:?}", e);
			      })
			      .ok()?;
		     let signature = raw_payload.using_encoded(|payload| C::sign(payload, public))?;
		     let address = account;
		     let (call, extra, _) = raw_payload.deconstruct();
		     Some((call, (sp_runtime::MultiAddress::Id(address), signature, extra)))
	   }
   }
   ```

   此代码段很长，但本质上，它说明了以下主要步骤：

   - 创建和准备 `SignedExtra` 类型的 `extra`，并设置各种检查器。
   - 基于传入的 `call` 和 `extra` 创建原始有效负载。
   - 使用帐户公钥对原始有效负载进行签名。
   - 将所有数据捆绑在一起并返回一个包含调用、调用者、其签名和任何已签名扩展数据的元组。

   您可以在[Substrate 代码库](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/runtime/src/lib.rs#L1239-L1279)中看到此代码的示例。

1. 在运行时实现 `SigningTypes` 和 `SendTransactionTypes` 以支持提交事务（无论它们是已签名的还是未签名的）。

   ```rust
   impl frame_system::offchain::SigningTypes for Runtime {
        type Public = <Signature as traits::Verify>::Signer;
        type Signature = Signature;
   }

   impl<C> frame_system::offchain::SendTransactionTypes<C> for Runtime
   where
        RuntimeCall: From<C>,
   {
        type Extrinsic = UncheckedExtrinsic;
        type OverarchingCall = RuntimeCall;
   }
   ```

   您可以在[Substrate 代码库](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/runtime/src/lib.rs#L1280-L1292)中看到此实现的示例。

### 添加用于签名事务的帐户

此时，您已准备好了模块以使用链下工作程序。
准备模块涉及以下步骤：

- 添加 `offchain_worker` 函数和发送已签名事务的相关逻辑。
- 向模块的 `Config` 特征添加 `CreateSignedTransaction` 和 `AuthorityId`。
- 添加 `crypto` 模块以描述模块将用于签名事务的帐户。

您还使用代码更新了运行时以支持链下工作程序和发送已签名事务。
更新运行时涉及以下步骤：

- 将 `AuthorityId` 添加到模块的运行时配置中。
- 实现 `CreateSignedTransaction` 特征和 `create_transaction()` 函数。
- 为来自 `frame_system` 模块的链下工作程序实现 `SigningTypes` 和 `SendTransactionTypes`。

但是，在模块链下工作程序能够提交已签名事务之前，您必须为链下工作程序指定至少一个帐户来使用。
要启用链下工作程序签名事务，您必须生成模块拥有的帐户密钥并将该密钥添加到节点密钥库。

完成此最后一步的方法有几种，您选择的方法可能因您是在开发模式下运行节点进行测试、使用自定义链规范还是部署到生产环境而异。

### 使用开发帐户

如果您在开发模式下运行节点（使用 `--dev` 命令行选项），则可以通过如下修改 `node/src/service.rs` 文件来手动生成和插入开发帐户的帐户密钥：

```rust
pub fn new_partial(config: &Configuration) -> Result <SomeStruct, SomeError> {

//...

  if config.offchain_worker.enabled {
  // Initialize seed for signing transaction using offchain workers. This is a convenience
  // so learners can see the transactions submitted simply running the node.
  // Typically these keys should be inserted with RPC calls to `author_insertKey`.
   	sp_keystore::SyncCryptoStore::sr25519_generate_new(
   		&*keystore,
   		node_template_runtime::pallet_your_ocw_pallet::KEY_TYPE,
   		Some("//Alice"),
   	).expect("Creating key with account Alice should succeed.");
   	}
}
```

此示例手动将 `Alice` 帐户的密钥添加到模块中定义的 `KEY_TYPE` 标识的密钥库中。
有关工作示例，请参阅此示例[service.rs](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/v2.0.0/node/src/service.rs#L87-L105)文件。

### 使用其他帐户

在生产环境中，您可以使用其他工具（例如 `subkey`）来生成链下工作程序专门使用的密钥。
为链下工作程序生成一个或多个密钥后，您可以通过以下方式将其添加到节点密钥库：

- 修改链规范文件的配置。
- 使用 `author_insertKey` RPC 方法传递参数。

例如，您可以使用[Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/rpc)、Polkadot-JS API 或 `curl` 命令来选择 `author_insertKey` 方法，并为要使用的帐户指定密钥类型、秘密短语和公钥参数：

![使用 `author_insertKey` 方法插入帐户](/media/images/docs/author_insertKey.png)

请注意，此示例中的密钥类型参数 `demo` 与链下工作程序模块中声明的 `KEY_TYPE` 匹配。

现在，您的模块已准备好从链下工作程序向链上发送已签名事务。

## 未签名事务

默认情况下，Substrate 会拒绝所有未签名事务。
要启用 Substrate 接受某些未签名事务，您必须为模块实现 `ValidateUnsigned` 特征。

尽管您必须实现 `ValidateUnsigned` 特征才能发送未签名事务，但这并不能保证**只有**链下工作程序才能发送事务。
您应该始终考虑恶意行为者发送这些事务以试图篡改链状态的后果。
未签名事务始终代表恶意用户可能利用的潜在攻击媒介，并且如果没有额外的安全措施，则不能假设链下工作程序是可靠的来源。

您永远不应该假设未签名事务只能由链下工作程序提交。
根据定义，**任何人都**可以提交它们。

### 配置模块

要启用链下工作程序发送未签名事务：

1. 在文本编辑器中打开模块的 `src/lib.rs` 文件。
2. 添加[`validate_unsigned`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#validate-unsigned-palletvalidate_unsigned-optional)宏。

	 例如：

   ```rust
   #[pallet::validate_unsigned]
   impl<T: Config> ValidateUnsigned for Pallet<T> {
   	type Call = Call<T>;

   		/// Validate unsigned call to this module.
   		///
   		/// By default unsigned transactions are disallowed, but implementing the validator
   		/// here we make sure that some particular calls (the ones produced by offchain worker)
   		/// are being whitelisted and marked as valid.
   		fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   		//...
   		}
   }
   ```

1. 按如下方式实现该特征：

   ```rust
   fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   	let valid_tx = |provide| ValidTransaction::with_tag_prefix("my-pallet")
   		.priority(UNSIGNED_TXS_PRIORITY) // please define `UNSIGNED_TXS_PRIORITY` before this line
   		.and_provides([&provide])
   		.longevity(3)
   		.propagate(true)
   		.build();
   	// ...
   }
   ```

2. 检查调用外部函数以确定是否允许调用，如果允许调用则返回 `ValidTransaction`，如果不允许调用则返回 `TransactionValidityError`。

	 例如：

   ```rust
   fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   	// ...
   	match call {
   		RuntimeCall::my_unsigned_tx { key: value } => valid_tx(b"my_unsigned_tx".to_vec()),
   		_ => InvalidTransaction::Call.into(),
   	}
   }
   ```

   在此示例中，用户只能调用不带签名的特定`my_unsigned_tx`函数。
	 如果还有其他函数，则调用它们需要已签名事务。

	 有关如何在模块中实现`ValidateUnsigned`的示例，请参阅[链下工作程序](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L301-L329)的代码。

3. 添加 `#[pallet::hooks]` 宏和 `offchain_worker` 函数以发送未签名事务，如下所示：

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	/// Offchain worker entry point.
   	fn offchain_worker(block_number: T::BlockNumber) {
   		let value: u64 = 10;
   		// This is your call to on-chain extrinsic together with any necessary parameters.
   		let call = RuntimeCall::unsigned_extrinsic1 { key: value };

   		// `submit_unsigned_transaction` returns a type of `Result<(), ()>`
   		//	 ref: https://paritytech.github.io/substrate/master/frame_system/offchain/struct.SubmitTransaction.html
   		SubmitTransaction::<T, Call<T>>::submit_unsigned_transaction(call.into())
   			.map_err(|_| {
   			log::error!("Failed in offchain_unsigned_tx");
   		});
   	}
   }
   ```

   此代码在`let call = ...`行中准备调用，使用[`SubmitTransaction::submit_unsigned_transaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/struct.SubmitTransaction.html)提交事务，并在传入的回调函数中执行任何必要的错误处理。

### 配置运行时

1. 通过向 `construct_runtime` 宏中添加 `ValidateUnsigned` 类型，在运行时为模块启用 `ValidateUnsigned` 特征。

   例如：

   ```rust
   construct_runtime!(
   	pub enum Runtime where
   		Block = Block,
   		NodeBlock = opaque::Block,
   		UncheckedExtrinsic = UncheckedExtrinsic
   	{
   		// ...
   		OcwPallet: pallet_ocw::{Pallet, Call, Storage, Event<T>, ValidateUnsigned},
   	}
   );
   ```

1. 如[发送已签名事务](#sending-signed-transactions)中所述，为运行时实现`SendTransactionTypes`特征。

   有关完整示例，请参阅[链下工作程序](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/examples/offchain-worker examples 模块。

## 已签名有效负载

发送带有已签名有效负载的未签名事务类似于发送未签名事务。
您需要：

- 为模块实现 `ValidateUnsigned` 特征。
- 使用此模块时，将 `ValidateUnsigned` 类型添加到运行时。
- 通过实现[`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)特征来准备要签名的结构（已签名有效负载）。
- 发送带有已签名有效负载的事务。

有关实现 `ValidateUnsigned` 特征以及将 `ValidateUnsigned` 类型添加到运行时的更多信息，您可以参考[发送未签名事务](#sending-unsigned-transactions)部分。

请记住，未签名事务始终代表潜在的攻击媒介，并且如果没有额外的安全措施，则不能假设链下工作程序是可靠的来源。
在大多数情况下，您应该实现限制性权限或其他逻辑来验证链下工作程序提交的事务是否有效。

以下代码示例说明了发送未签名事务和发送带有已签名有效负载的未签名事务之间的区别。

要使您的数据结构可签名：

1. 实现[`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)。

   例如：

   ```rust
   #[derive(Encode, Decode, Clone, PartialEq, Eq, RuntimeDebug, scale_info::TypeInfo)]
   pub struct Payload<Public> {
   	number: u64,
   	public: Public,
   }

   impl<T: SigningTypes> SignedPayload<T> for Payload<T::Public> {
   	fn public(&self) -> T::Public {
   	self.public.clone()
   }
   }
   ```

  有关已签名有效负载的示例，请参阅[链下工作程序](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L348-L361)的代码。

1. 在 `offchain_worker` 函数中，调用签名者，然后调用发送事务的函数：

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	/// Offchain worker entry point.
   	fn offchain_worker(block_number: T::BlockNumber) {
   		let value: u64 = 10;

   		// Retrieve the signer to sign the payload
   		let signer = Signer::<T, T::AuthorityId>::any_account();

   		// `send_unsigned_transaction` is returning a type of `Option<(Account<T>, Result<(), ()>)>`.
   		//	 The returned result means:
   		//	 - `None`: no account is available for sending transaction
   		//	 - `Some((account, Ok(())))`: transaction is successfully sent
   		//	 - `Some((account, Err(())))`: error occurred when sending the transaction
   		if let Some((_, res)) = signer.send_unsigned_transaction(
   			// this line is to prepare and return payload
   			|acct| Payload { number, public: acct.public.clone() },
   			|payload, signature| RuntimeCall::some_extrinsics { payload, signature },
   		) {
   			match res {
   				Ok(()) => log::info!("unsigned tx with signed payload successfully sent.");
   				Err(()) => log::error!("sending unsigned tx with signed payload failed.");
   			};
   		} else {
   			// The case of `None`: no account is available for sending
   			log::error!("No local account available");
   		}
   	}
   }
   ```

   此代码检索`签名者`，然后使用两个函数闭包调用`send_unsigned_transaction()`。
   第一个函数闭包返回要使用的有效负载。
   第二个函数闭包返回带有传入有效负载和签名的链上调用。
   此调用返回`Option<(Account<T>, Result<(), ()>)>`结果类型，以允许以下结果：

   - 如果没有可用于发送事务的帐户，则为`None`。
   - 如果事务已成功发送，则为`Some((account, Ok(())))`。
   - 如果发送事务时发生错误，则为`Some((account, Err(())))`。

2. 检查提供的签名是否与用于对有效负载进行签名的公钥匹配：

   ```rust
   #[pallet::validate_unsigned]
   impl<T: Config> ValidateUnsigned for Pallet<T> {
   	type Call = Call<T>;

   	fn validate_unsigned(_source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   		let valid_tx = |provide| ValidTransaction::with_tag_prefix("ocw-demo")
   			.priority(UNSIGNED_TXS_PRIORITY)
   			.and_provides([&provide])
   			.longevity(3)
   			.propagate(true)
   			.build();

   		match call {
   			RuntimeCall::unsigned_extrinsic_with_signed_payload {
   			ref payload,
   			ref signature
   			} => {
   			if !SignedPayload::<T>::verify::<T::AuthorityId>(payload, signature.clone()) {
   				return InvalidTransaction::BadProof.into();
   			}
   			valid_tx(b"unsigned_extrinsic_with_signed_payload".to_vec())
   			},
   			_ => InvalidTransaction::Call.into(),
   		}
   	}
   }
   ```

   此示例使用[`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)来验证有效负载中的公钥是否与提供的公钥具有相同的签名。
	 但是，您应该注意，示例中的代码仅检查提供的`signature`对于`payload`中包含的`public`密钥是否有效。
	 此检查不会验证签名者是否是链下工作程序或是否有权调用指定的函数。
	 此简单的检查不会阻止未经授权的行为者使用已签名有效负载来修改状态。

	 有关此代码的工作示例，请参阅[链下函数调用](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L508-L536)以及[`ValidateUnsigned`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L305-L329)的实现。

## 下一步

本教程提供了如何使用链下工作程序发送链上存储事务的简单示例。
要了解更多信息，请浏览以下资源：

- [链下操作](/learn/offchain-operations/)
- [链下工作程序示例](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/examples/offchain-worker)
- [链下工作程序演示](https://github.com/jimmychu0807/substrate-offchain-worker-demo/tree/v2.0.0/pallets/ocw)
