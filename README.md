if (!window._origSendMouse) {
    window._origSendMouse = window.CORE?.net?.sendMouseMove;
    
    if (window.CORE?.net) {
        window.CORE.net.sendMouseMove = function(...args) {
            if (window.gamePaused) {
                // Отправляем фальшивые координаты (центр экрана)
                const fakePacket = new (window.At || class {
                    constructor() {
                        this._b = [];
                        this.writer = true;
                    }
                    setUint8(v) { this._b.push(v); return this; }
                    setFloat64(v) {
                        const buf = new ArrayBuffer(8);
                        new DataView(buf).setFloat64(0, v, true);
                        for (let i = 0; i < 8; i++) this._b.push(buf[i]);
                        return this;
                    }
                    setUint32(v) {
                        this._b.push(v & 0xff, (v >> 8) & 0xff, (v >> 16) & 0xff, (v >> 24) & 0xff);
                        return this;
                    }
                    build() { return new Uint8Array(this._b); }
                })();
                fakePacket.setUint8(16); // MOUSE
                fakePacket.setFloat64(0); // x = 0
                fakePacket.setFloat64(0); // y = 0
                fakePacket.setUint32(0);
                window.CORE.net.send(fakePacket);
                return;
            }
            return window._origSendMouse.apply(this, args);
        };
        console.log('ВКЛ');
    } 
}
