### Lit Lifecycle
```javascript
    <my-element name="atom" />
```
#### Standard custom element lifecycle (inherite from Web Components standard)
- constructor()
- connectedCallback()
- disconnectedCallback() 
- attributeChangedCallback()
- adoptedCallback()

###### Use cases

- constructor()
    ```javascript
    constructor() {
        super();
        this.foo = 'foo';
        this.bar = 'bar';
    }
    ```
    
- connectedCallback()

    ```javascript
    // example 1
    override connectedCallback(): void {
        super.connectedCallback()
        window.addEventListener('keydown', this._handleKeydown);
    }
    // example 2
    override connectedCallback (): void {
        super.connectedCallback();
        if (this.canvas) {
            this.createChart();
        }
    }
    ```
- disconnectedCallback() 
    ```javascript
     // example 1
    override disconnectedCallback(): void {
        super.disconnectedCallback()
        window.removeEventListener('keydown', this._handleKeydown);
    }
     // example 2
     override disconnectedCallback (): void {
        super.disconnectedCallback();
        this.destroyChart();
    }
    ```


- attributeChangedCallback()
    - Invoked when one of the element’s `observedAttributes` changes.
    ```javascript
    override attributeChangedCallback (name: string, oldValue: string | null, newValue: string | null): void {
        super.attributeChangedCallback(name, oldValue, newValue);
        console.log('attributeChangedCallback', {name, oldValue, newValue});
    }
    ```
    
- adoptedCallback()
    - Invoked when a component is moved to a new document.
    
    
    ```javascript

    const iframe = document.querySelector('iframe');
    const iframeImages = iframe.contentDocument.querySelectorAll('img');
    const newParent = document.getElementById('images');

    iframeImages.forEach(function(imgEl) {
        newParent.appendChild(document.adoptNode(imgEl));
    });


    adoptedCallback() {
      
    }
    ```

#### Reactive update cycle
###### Triggering an update
![Pre-Update](https://lit.dev/images/docs/components/update-1.jpg)
![Pre-Update](https://lit.dev/images/docs/components/update-2.jpg)
###### Performing an update
![Update](https://lit.dev/images/docs/components/update-3.jpg)
###### Completing an update
![Post-Update](https://lit.dev/images/docs/components/update-4.jpg)

- Triggering an update
    - hasChanged()
    ```javascript
    @property({type: Number,
    // only update for odd values of newVal.
    hasChanged(newVal: number, oldVal: number) {
        return newVal % 2 == 1;
    }})
    count = 0;
    ```

    - requestUpdate()
    ```javascript
    if (value !== oldValue) {
        this.requestUpdate('data', oldValue);
    }
    ```


- Performing an update
    - shouldUpdate()
    ```javascript
    override shouldUpdate (changedProperties: PropertyValues): boolean {
        const isOpened = this.opened;
        const isClosed = !this.opened;
        const opening = changedProperties.has('opened') && isOpened;
        const closing = changedProperties.has('opened') && isClosed;
        
        return opening || closing || !this.hasUpdated || changedProperties.size === 0;
    }
    ```

    - willUpdate()
    ```javascript
    // example 1
    protected willUpdate (changedProperties: PropertyValues): void {
        if (changedProperties.has('toggles') || changedProperties.has('active')) {
            if (this.toggles) {
                this.setAttribute('aria-pressed', String(this.active));
            }
            else {
                this.removeAttribute('aria-pressed');
            }
        }
    }

    // example 2
    protected willUpdate(changedProperties: PropertyValues): void {
        // only need to check changed properties for an expensive computation.
        if (changedProperties.has('firstName') || changedProperties.has('lastName')) {
            this.sha = computeSHA(`${this.firstName} ${this.lastName}`);
        }
    }
    ```
    - update()
        - Without a super call, the element’s attributes and template will not update.
        - Reflects property values to attributes and calls render()
        - Generally, you should not need to implement this method.
    - render()
        - return TemplateResult
    ```javascript
    // example 1
    protected render (): TemplateResult {
        return html`
            <h1>Hello, ${this.name}!</h1>
        `;
    }

    // example2
    protected render (): TemplateResult {
        return html`SHA: ${this.sha}`;
    }
    ```
- Completing an update
    - firstUpdated()
        - Called after the component's DOM has been updated the first time
    ```javascript
    // example 1
    protected firstUpdated (changedProperties: PropertyValues): void {
        super.firstUpdated(changedProperties);
        this.setAttribute('aria-modal', String(!this.noInteractionLock));
    }

    // example 2
    protected firstUpdated (changeProperties: PropertyValues): void {
        super.firstUpdated(changeProperties);
        this.addEventListener('keydown', this.onKeyDown);
    }
    ```
    - updated()
    ```javascript
    protected updated (changedProperties: PropertyValues): void {
        super.updated(changedProperties);
        if (changedProperties.has('sidebarWidth')) {
            this.updateVariable('--sidebar-width', this.sidebarWidth);
        }
    }
    ```
    - updateComplete
    ```javascript
    private async _onClick() {
        this.count++;
        await this.updateComplete;
        this.dispatchEvent(new CustomEvent('count-changed'));
    }
    ```
- Implementing additional customization
    - performUpdate()
    ```javascript
    override async performUpdate (): Promise<void> {
        await new Promise((resolve) => setTimeout(() => resolve({}), 200));
        void super.performUpdate(); 
    }
    ```
    - hasUpdated
        - The hasUpdated property returns true if the component has updated at least once
        - perform work only if the component has not yet updated
    - getUpdateComplete()
        - can override to wait child element
    ```javascript
    override async getUpdateComplete() {
        const result = await super.getUpdateComplete();
        await this._myChild.updateComplete;
        return result;
    }
    ```
