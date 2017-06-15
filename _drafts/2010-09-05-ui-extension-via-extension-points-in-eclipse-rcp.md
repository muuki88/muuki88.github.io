---
author: wp_admin
comments: false
date: 2010-09-05 20:34:35+00:00
layout: post
link: http://mukis.de/pages/ui-extension-via-extension-points-in-eclipse-rcp/
slug: ui-extension-via-extension-points-in-eclipse-rcp
title: UI Extension via Extension Points in Eclipse RCP
wordpress_id: 65
categories:
- Tutorials
---

Eclipse has a powerful mechanism to allow Plugins to contribute to the UI: **Extensions** and **Extension Points**. There are a lot of excellent tutorials like [Eclipse Extensions](http://www.vogella.de/articles/EclipseExtensionPoint/article.html) by[ Lars Vogel](http://www.vogella.de/) on the internet. However this little tutorial is about how to contribute  to an Editor (in this case an additional TabItem).




**1. The Extension Interface**




First we have to create an Interface which the Extension has to implement. To create an additional Tab in an Editor I created an Interface like this:



    
    public interface IEditorTabExtension {
    
    	/**
    	 * Is called to create the tab control
    	 * @param parent
    	 * @return Control - The created Control
    	 */
    	public Control createContents(Composite parent);
    
    	/**
    	 * Should be called by the doSave method in
    	 * the root EditorPart
    	 *
    	 * @param monitor
    	 */
    	public void doSave(IProgressMonitor monitor);
    
    	/**
    	 * Call-by-Reference dirty boolean. Indicates
    	 * if changes were made.
    	 *
    	 * @param dirty
    	 */
    	public void setDirty(Boolean dirty);
    
    	/**
    	 *
    	 * @return Name for the Tab
    	 */
    	public String getName();
    }
    




**2. Create the Extension Point**




First we create an Extension Point in the plugin.xml via the plugin.xml Editor.




[![Create Extension Point](http://mukis.de/pages/wp-content/uploads/2010/09/tutorial01.png)](http://mukis.de/pages/?attachment_id=76)




The Extension-Schema Editor should now open automatically. Otherwise there's a button.  

Add a new Element and call it "tab". Now add a new attribute and name it "class". Type should be "java" and Implements  

our IEditorTabExtension. Don't forget to create a new attribute _Choice _ in the "extension" element. And in there an  

"Tab" entry. Now it should look like this:




[![Extension Point Elements](http://mukis.de/pages/wp-content/uploads/2010/09/tutorial021-300x264.png)](http://mukis.de/pages/wp-content/uploads/2010/09/tutorial021.png)




**3. Create an Extension and provide it**




Our Plugin can not only provide an extension point, it provides an extension too. Feel free to  

implement the Interface with an UI you like. To register this Extension open the plugin.xml  

and the _Extensions _Tab. Add our new Extension Point _de.mukis.editor.EditorTabExtension_.  

Should look like this:




[![Provide Extension](http://mukis.de/pages/wp-content/uploads/2010/09/tutorial03-300x264.png)](http://mukis.de/pages/wp-content/uploads/2010/09/tutorial03.png)




**4. Evaluate Contribs and add it to the Editor**



    
    private IEditorTabExtension[] extensions;
    
    	@Override
    	public void doSave(IProgressMonitor monitor) {
    		dirty = false;
    		for(IEditorTabExtension e : extensions)
    			e.doSave(monitor);
    		firePropertyChange(IWorkbenchPartConstants.PROP_DIRTY);
    	}
    
    	@Override
    	public void createPartControl(Composite parent) {
    		folder = new TabFolder(parent, SWT.BORDER);
    
    		extensions = evaluateTabContribs();
    		for (IEditorTabExtension e : extensions) {
    			TabItem tab = new TabItem(folder, SWT.BORDER);
    			tab.setText(e.getName());
    			tab.setControl(e.createContents(folder));
    			System.out.println("Tab added");
    		}
    
    	}
    
     private IChildEditorTabExtension[] evaluateTabContribs() {
    		IConfigurationElement[] config = Platform.getExtensionRegistry()
    				.getConfigurationElementsFor(TAB_ID);
    		final LinkedList list = new LinkedList();
    		try {
    			for(IConfigurationElement e : config) {
    				System.out.println("Evaluation extension");
    				final Object o = e.createExecutableExtension("class");
    				if(o instanceof IEditorTabExtension) {
    					ISafeRunnable runnable = new ISafeRunnable() {
    
    						@Override
    						public void handleException(Throwable exception) {
    							System.out.println("Exception in Tab");
    						}
    
    						@Override
    						public void run() throws Exception {
    							IEditorTabExtension tab = (IEditorTabExtension)o;
    							list.add(tab);
    							System.out.println("Extension detected: " + tab.getName());
    						}
    					};
    					SafeRunner.run(runnable);
    				}
    			}
    		} catch(CoreException ex) {
    			System.out.println(ex.getMessage());
    		}
    		return list.toArray(new IChildEditorTabExtension[list.size()]);
    	}
    




This is very basic. The isDirty flag solution isn't very smart. We use the Call-by-Reference effect  

to provide a "global" Boolean.




Thanks to Lars Vogel's tutorials which inspired me to to my own stuff  and have been used  

for this tutorial.



