# Uniswap Routing API - Immutable zkEVM

> **Note:** This is a Labrys fork of the [Uniswap Routing API](https://github.com/Uniswap/routing-api), customized for **Immutable zkEVM testnet** deployment. It provides V3-only routing functionality for a single chain.

This repository contains a routing API for the Uniswap V3 protocol, deployed as an AWS Lambda function that uses `@Labrys-Group/smart-order-router` to find the most efficient way to swap token A for token B.

## Key Differences from Upstream

- **Single Chain Only:** Supports only Immutable zkEVM Testnet (Chain ID: 13473)
- **V3 Only:** V2 and V4 protocol support has been removed
- **Simplified Configuration:** No Tenderly or Alchemy integrations required
- **Custom Dependencies:** Uses `@Labrys-Group/smart-order-router` package

## Supported Chain

This API exclusively supports:

- **Immutable zkEVM Testnet** (Chain ID: `13473`)

## Prerequisites

### Required Tools

- Node.js 18.x
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) (configured with your credentials)
- [Task](https://taskfile.dev/) (optional, for automated tasks)

### GitHub Package Authentication

This project uses packages from the `@Labrys-Group` GitHub package repository. You must authenticate before running `npm install`.
Create a GitHub Personal Access Token with `read:packages` scope and configure npm:

**Option 1: Using Taskfile (Recommended)**

```bash
task package:login
```

## Development Setup

1. **Clone the repository**

   ```bash
   git clone git@github.com:Labrys-Group/uniswap-routing-api.git
   cd uniswap-routing-api
   ```

2. **Authenticate to GitHub Packages** (see above)

3. **Create environment configuration**

   ```bash
   cp .env.example .env
   ```

   Then edit `.env` and configure the required variables (see Environment Variables section below).

4. **Install dependencies and build**
   ```bash
   npm install
   npm run build
   ```

## Environment Variables

### Required Variables

```bash
# GitHub personal access token for authentication to access @Labrys-Group packages
GITHUB_PACKAGE_TOKEN=ghp_your_token_here

# RPC endpoint for Immutable zkEVM testnet
ZK_EVM_TESTNET_RPC=https://rpc.testnet.immutable.com

# Your deployed V3 subgraph URL
V3_SUBGRAPH_URL=https://your-subgraph-url/graphql

# Application version
VERSION=1

# Node.js options for source maps
NODE_OPTIONS=--enable-source-maps

# AWS Region (for DynamoDB and Lambda)
AWS_REGION=ap-southeast-2
```

### Optional Variables (Caching)

For production deployments, DynamoDB caching is recommended:

```bash
# Routes cache tables
ROUTES_TABLE_NAME=RoutesDB
CACHED_ROUTES_TABLE_NAME=RouteCachingDB
ROUTES_CACHING_REQUEST_FLAG_TABLE_NAME=RoutesDbCacheReqFlagDB

# V3 pools cache table
CACHED_V3_POOLS_TABLE_NAME=V3PoolsCachingDB
```

### Optional Variables (Debug)

```bash
# Secret for enabling debug routing config
UNICORN_SECRET=your-random-secret-string
```

## Deployment

### Option 1: Terraform (Recommended)

For comprehensive deployment using Terraform, refer to the [Terraform Deployment Guide](TERRAFORM_DEPLOYMENT.md). This guide covers:

- Complete infrastructure setup (Lambda, DynamoDB, API Gateway)
- IAM roles and permissions
- Environment configuration
- Monitoring and logging

### Option 2: GitHub Actions (CI/CD)

This repository includes automated deployment via GitHub Actions:

**Workflow:** `.github/workflows/deploy.yml`

**Required GitHub Secrets:**

- `AWS_ROLE_ARN` - IAM role ARN for deployment
- `AWS_REGION` - AWS region (e.g., ap-southeast-2)
- `LAMBDA_FUNCTION_NAME` - Name of your Lambda function

**Required GitHub Variables:**

- `UNI_GRAPHQL_ENDPOINT` - Your V3 subgraph URL
- `UNI_GRAPHQL_HEADER_ORIGIN` - GraphQL header origin

The workflow automatically builds and deploys on push to the main branch.

### Lambda Configuration

The deployed Lambda function uses:

- **Runtime:** Node.js 18.x
- **Handler:** `lib/handlers/index.quoteHandler`
- **Memory:** 5120 MB
- **Timeout:** 30 seconds

## API Usage

### Endpoint

```
POST /quote
```

### Request Format

```bash
curl -X POST 'https://your-api-gateway-url/quote' \
  -H 'Content-Type: application/json' \
  -d '{
    "tokenInAddress": "0x...",
    "tokenInChainId": 13473,
    "tokenOutAddress": "0x...",
    "tokenOutChainId": 13473,
    "amount": "1000000000000000000",
    "type": "exactIn",
    "recipient": "0x...",
    "slippageTolerance": "0.5",
    "deadline": 1234567890
  }'
```

### Key Parameters

- `tokenInChainId` and `tokenOutChainId` **must** be `13473` (Immutable zkEVM Testnet)
- `amount` is in wei (for exactIn) or token units (for exactOut)
- `type` can be `exactIn` or `exactOut`
- `slippageTolerance` is a percentage (e.g., "0.5" = 0.5%)

### Response Format

```json
{
  "methodParameters": {
    "calldata": "0x...",
    "value": "0x00",
    "to": "0x..."
  },
  "blockNumber": "12345678",
  "amount": "1000000000000000000",
  "amountDecimals": "1.0",
  "quote": "999000000000000000",
  "quoteDecimals": "0.999",
  "quoteGasAdjusted": "998000000000000000",
  "quoteGasAdjustedDecimals": "0.998",
  "gasUseEstimateQuote": "1000000000000000",
  "gasUseEstimateQuoteDecimals": "0.001",
  "gasUseEstimate": "150000",
  "gasUseEstimateUSD": "0.50",
  "gasPriceWei": "1000000000",
  "route": [...]
}
```

## Testing

### Unit Tests

```bash
npm run test:unit
```

Watch mode:

```bash
npm run test:unit:watch
```

### Integration Tests

Integration tests run against a local DynamoDB instance (requires JDK 8):

```bash
npm run test:integ
```

### End-to-End Tests

E2E tests fetch quotes from your deployed API and execute swaps on a Hardhat fork:

1. Update `.env` with your API URL and archive node:

   ```bash
   UNISWAP_ROUTING_API='https://your-api-gateway-url'
   ARCHIVE_NODE_RPC='https://your-archive-node-rpc'
   ```

2. Run the tests:
   ```bash
   npm run test:e2e
   ```

**Note:** Test files may not be present in all forks. Check for the existence of test directories before running.

## Available Scripts

- `npm run build` - Build the TypeScript code
- `npm run build:clean` - Clean and rebuild
- `npm run lint` - Run ESLint
- `npm run lint:fix` - Auto-fix linting issues
- `npm run test:unit` - Run unit tests
- `npm run test:integ` - Run integration tests
- `npm run test:e2e` - Run end-to-end tests

## Taskfile Commands

If you have [Task](https://taskfile.dev/) installed:

- `task package:login` - Authenticate to GitHub packages
- `task build` - Build the project
- `task test` - Run all tests

Run `task --list` to see all available tasks.

## Architecture

This API uses:

- **AWS Lambda** - Serverless compute for quote generation
- **API Gateway** - HTTP API endpoint
- **DynamoDB** - Caching for routes and pool data (optional but recommended)
- **S3** - Token list and pool cache storage (optional)

The quote generation flow:

1. API Gateway receives POST request
2. Lambda handler validates request and extracts parameters
3. Smart Order Router queries V3 subgraph for pool data
4. Router calculates optimal V3-only route
5. Response includes calldata ready for on-chain execution

## Monitoring

The Lambda function emits logs to CloudWatch. Key metrics to monitor:

- Invocation count
- Error rate
- Duration (p50, p99)
- Throttles

## Troubleshooting

### Common Issues

**"Cannot authenticate to @Labrys-Group packages"**

- Run `task package:login` or manually configure npm authentication
- Ensure your GitHub token has `read:packages` scope

**"Chain ID not supported"**

- This API only supports chain ID 13473 (Immutable zkEVM Testnet)
- Ensure both `tokenInChainId` and `tokenOutChainId` are set to 13473

**"Insufficient liquidity"**

- Check that V3 pools exist for your token pair on Immutable zkEVM testnet
- Verify your V3 subgraph is properly indexed and accessible

**"Subgraph query failed"**

- Verify `V3_SUBGRAPH_URL` is correct and accessible
- Check subgraph sync status

## Contributing

This is a fork maintained by Labrys Group for Immutable zkEVM. For contributions:

1. Create a feature branch
2. Make your changes with tests
3. Submit a pull request

## Related Documentation

- [Terraform Deployment Guide](TERRAFORM_DEPLOYMENT.md)
- [Immutable zkEVM Documentation](https://docs.immutable.com/)
- [Upstream Uniswap Routing API](https://github.com/Uniswap/routing-api)

## License

See [LICENSE](LICENSE) file for details.

# Routing API - New Chain Support Checklist

This document outlines all the changes required to add support for a new chain in the routing-api.

> **Note:** The routing-api depends on `@uniswap/smart-order-router`. You must first add chain support there and publish a new version before the routing-api will work. See the smart-order-router's `NEW_CHAIN_SUPPORT.md` for those steps.

## 1. `lib/constants/` - Chain ID Constant

Create a new constants file for your chain:

```typescript
// lib/constants/my-chain.ts
export const MY_CHAIN_ID = 12345
```

## 2. `lib/handlers/injector-sor.ts` - Supported Chains

```typescript
import { MY_CHAIN_ID } from '../constants/my-chain'

// Add to SUPPORTED_CHAINS array
export const SUPPORTED_CHAINS: ChainId[] = [
  // ... existing chains
  MY_CHAIN_ID,
]
```

## 3. `lib/util/onChainQuoteProviderConfigs.ts` - Quote Provider Config

This is the most critical file. Add your chain to ALL config maps:

```typescript
import { MY_CHAIN_ID } from '../constants/my-chain'

// 1. RETRY_OPTIONS
export const RETRY_OPTIONS: { [chainId: number]: AsyncRetry.Options | undefined } = {
  // ... existing entries
  [MY_CHAIN_ID]: {
    retries: 2,
    minTimeout: 100,
    maxTimeout: 1000,
  },
}

// 2. OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS (for each protocol you support)
export const OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS = {
  [Protocol.V3]: {
    // ... existing entries
    [MY_CHAIN_ID]: {
      multicallChunk: 80,
      gasLimitPerCall: 1_200_000,
      quoteMinSuccessRate: 0.1,
    },
  },
  // Add to V2, V4, MIXED if supported
}

// 3. NON_OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS (same structure)
export const NON_OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS = {
  [Protocol.V3]: {
    // ... existing entries
    [MY_CHAIN_ID]: {
      multicallChunk: 80,
      gasLimitPerCall: 1_200_000,
      quoteMinSuccessRate: 0.1,
    },
  },
}

// 4. GAS_ERROR_FAILURE_OVERRIDES
export const GAS_ERROR_FAILURE_OVERRIDES: { [chainId: number]: FailureOverrides } = {
  // ... existing entries
  [MY_CHAIN_ID]: {
    gasLimitOverride: 3_000_000,
    multicallChunk: 45,
  },
}

// 5. SUCCESS_RATE_FAILURE_OVERRIDES
export const SUCCESS_RATE_FAILURE_OVERRIDES: { [chainId: number]: FailureOverrides } = {
  // ... existing entries
  [MY_CHAIN_ID]: {
    gasLimitOverride: 3_000_000,
    multicallChunk: 45,
  },
}

// 6. BLOCK_NUMBER_CONFIGS
export const BLOCK_NUMBER_CONFIGS: { [chainId: number]: BlockNumberConfig } = {
  // ... existing entries
  [MY_CHAIN_ID]: {
    baseBlockOffset: 0,
    rollback: {
      enabled: false,
    },
  },
}
```

## 4. `lib/cron/cache-config.ts` - Subgraph & Protocol Config

```typescript
import { MY_CHAIN_ID } from '../constants/my-chain'

// Subgraph URL override (if using custom subgraph)
export const v3SubgraphUrlOverride = (chainId: number) => {
  switch (chainId) {
    case MY_CHAIN_ID:
      return process.env.V3_SUBGRAPH_URL || ''
    default:
      return undefined
  }
}

// Protocol configuration
export const chainProtocols: ChainProtocol[] = [
  // ... existing entries
  {
    protocol: Protocol.V3,
    chainId: MY_CHAIN_ID,
    timeout: 90000,
    provider: new V3SubgraphProvider(
      MY_CHAIN_ID,
      3, // retries
      90000, // timeout
      true, // useRawResults
      v3TrackedEthThreshold,
      v3UntrackedUsdThreshold,
      v3SubgraphUrlOverride(MY_CHAIN_ID)
    ),
  },
]
```

## 5. `lib/config/rpcProviderProdConfig.json` - RPC Provider Config

```json
[
  {
    "chainId": 12345,
    "useMultiProviderProb": 1,
    "latencyEvaluationSampleProb": 0,
    "healthCheckSampleProb": 0,
    "providerInitialWeights": [1],
    "providerUrls": ["MY_CHAIN_RPC"],
    "providerNames": ["MY_CHAIN_RPC"]
  }
]
```

## 6. `lib/rpc/utils.ts` - RPC URL Resolution

```typescript
// In the switch statement for buildProviderUrl()
case 'MY_CHAIN_RPC': {
  return value  // Returns the env var value directly
}
```

## 7. Environment Variables

Set these in your deployment (AWS Lambda, etc.):

```bash
# RPC endpoint
WEB3_RPC_12345=https://rpc.mychain.com

# Or if using named provider in rpcProviderProdConfig.json
MY_CHAIN_RPC=https://rpc.mychain.com

# Subgraph URL (if applicable)
V3_SUBGRAPH_URL=https://api.thegraph.com/subgraphs/name/...
```

## 8. `lib/handlers/injector-sor.ts` - Custom OnChainQuoteProvider (Optional)

If your chain needs special handling (e.g., V3-only support), add custom logic:

```typescript
// Custom quote provider for specific chain
if (chainId === (MY_CHAIN_ID as ChainId)) {
  quoteProvider = new OnChainQuoteProvider(
    chainId,
    provider,
    multicall2Provider,
    RETRY_OPTIONS[chainId] || { retries: 2, minTimeout: 100, maxTimeout: 1000 },
    (optimisticCachedRoutes, _protocol) => {
      const protocol = Protocol.V3
      return optimisticCachedRoutes
        ? OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS[protocol][chainId] || {
            multicallChunk: 80,
            gasLimitPerCall: 1_200_000,
            quoteMinSuccessRate: 0.1,
          }
        : NON_OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS[protocol][chainId] || {
            multicallChunk: 80,
            gasLimitPerCall: 1_200_000,
            quoteMinSuccessRate: 0.1,
          }
    },
    (_protocol) => GAS_ERROR_FAILURE_OVERRIDES[chainId] || { gasLimitOverride: 3_000_000, multicallChunk: 45 },
    (_protocol) => SUCCESS_RATE_FAILURE_OVERRIDES[chainId] || { gasLimitOverride: 3_000_000, multicallChunk: 45 },
    (_protocol) => BLOCK_NUMBER_CONFIGS[chainId] || { baseBlockOffset: 0, rollback: { enabled: false } },
    (_useMixedRouteQuoter, _mixedRouteContainsV4Pool, _protocol) => QUOTER_V2_ADDRESSES[chainId]
  )
}
```

---

## Summary

For a new chain in the routing-api:

1. **Create chain constant** in `lib/constants/`
2. **Add to SUPPORTED_CHAINS** in `injector-sor.ts`
3. **Configure quote provider params** in `onChainQuoteProviderConfigs.ts`:
   - RETRY_OPTIONS
   - OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS
   - NON_OPTIMISTIC_CACHED_ROUTES_BATCH_PARAMS
   - GAS_ERROR_FAILURE_OVERRIDES
   - SUCCESS_RATE_FAILURE_OVERRIDES
   - BLOCK_NUMBER_CONFIGS
4. **Configure subgraph/protocol** in `cache-config.ts`
5. **Add RPC config** in `rpcProviderProdConfig.json`
6. **Add RPC URL resolver** in `rpc/utils.ts`
7. **Set environment variables** for RPC and subgraph URLs

## Common Issues

### `normalizedChunk: null` Error

**Cause:** Chain not in quote provider config maps.
**Fix:** Add chain to ALL maps in `onChainQuoteProviderConfigs.ts`.

### `"methodParameters.to" is not allowed to be empty`

**Cause:** SwapRouter02 address not configured in smart-order-router.
**Fix:** Add `SWAP_ROUTER_02_ADDRESSES` entry in smart-order-router's `addresses.ts`.

### Wrong property names in fallback objects

**Correct properties:**

- `multicallChunk` (not `batchSize`)
- `gasLimitPerCall`
- `quoteMinSuccessRate` (not `dropUnexecutedFetches`)

## Dependency Flow

```
smart-order-router (npm package)
        ↓
   routing-api (imports @uniswap/smart-order-router)
        ↓
   AWS Lambda deployment
```

Always update and publish smart-order-router first, then update routing-api's package.json to use the new version.
