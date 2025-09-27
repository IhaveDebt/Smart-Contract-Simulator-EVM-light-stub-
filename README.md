/**
 * Smart Contract Simulator (evm_simulator.ts)
 *
 * Minimal EVM-like environment: register contracts (JS functions) and call them with state.
 * Good for unit testing and experiments (not a real EVM).
 *
 * Usage: ts-node src/evm_simulator.ts
 */
type Contract = { name: string; code: (state:any, method:string, args:any[]) => any };

class Runtime {
  state: Record<string, any> = {};
  contracts: Record<string, Contract> = {};
  deploy(c: Contract) { this.contracts[c.name] = c; this.state[c.name] = {}; }
  call(name: string, method: string, args: any[]) {
    const c = this.contracts[name];
    if (!c) throw new Error('contract not found');
    return c.code(this.state[name], method, args);
  }
}

// Demo contract: simple token
const token: Contract = {
  name: 'SimpleToken',
  code: (s, method, args) => {
    s.balances = s.balances || {};
    if (method === 'mint') { const [who, amt] = args; s.balances[who] = (s.balances[who]||0)+amt; return true; }
    if (method === 'balanceOf') { const [who] = args; return s.balances[who]||0; }
    return null;
  }
};

(function demo(){
  const rt = new Runtime();
  rt.deploy(token);
  rt.call('SimpleToken','mint',['alice',100]);
  console.log('alice balance', rt.call('SimpleToken','balanceOf',['alice']));
})();
