# API



Com a pasta e o terminal aberto:

```
yarn init -y
```

se nao tiver instalado:
```
npm install --global yarn 
```

## Mais 
```
yarn add -D typescript nodemon ts-node @types/express @types/node
```


## Biblioteca de produção
```
yarn add express pg typeorm dotenv reflect-metadata
```



# o arquivo packege
```
{
  "name": "API-REST-TYPESCRIPT",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",

  "scripts": {
    "dev": "nodemon --exec ts-node ./src/index.ts"
  },


  "devDependencies": {
    "@types/express": "^5.0.6",
    "@types/node": "^25.8.0",
    "nodemon": "^3.1.14",
    "ts-node": "^10.9.2",
    "typescript": "^6.0.3"
  },
  "dependencies": {
    "dotenv": "^17.4.2",
    "express": "^5.2.1",
    "pg": "^8.20.0",
    "reflect-metadata": "^0.2.2",
    "typeorm": "^0.3.29"
  }
}
```

### +

```
npx tsc --init
```


### ts.config.json
```
{
	"compilerOptions": {
		"target": "es2018",
		"lib": ["es5", "es6", "ES2018"],
		"experimentalDecorators": true,
		"emitDecoratorMetadata": true,
		"module": "commonjs",
		"moduleResolution": "node",
		"resolveJsonModule": true,
		"allowJs": true,
		"outDir": "./dist",
		"esModuleInterop": true,
		"forceConsistentCasingInFileNames": true,
		"strict": true,
		"noImplicitAny": true,
		"strictPropertyInitialization": false
	},
	"include": ["src/**/*"],
	"exclude": ["node_modules", "dist"],
	"ts-node": {
		"files": true
	}
}

```

